---
title: "Android"
linkTitle: "Android"
weight: 50
description: "go-widgets/android : un APK installable dont toute l'interface est mise en page et peinte par un processus Go CGO_ENABLED=0."
tags: [surfaces, android]
---

Android est la seule plateforme où un processus sans CGO ne peut pas posséder de fenêtre : tout
chemin vers une surface dessinable passe par JNI, et JNI exige cgo. Il n'existe pas
de protocole réseau à parler à la place, comme en ont X11 et Wayland.

[`go-widgets/android`](https://github.com/go-widgets/android) se compose donc de deux
processus :

| | |
|---|---|
| **Hôte Java** (`host/`) | possède l'`Activity`, la `SurfaceView`, le tactile, les touches et le cycle de vie. Copie des pixels ; ne sait rien des widgets. |
| **Application Go** (`cmd/gwapp`) | un exécutable `CGO_ENABLED=0 GOOS=android` ordinaire. Possède la mise en page, les widgets, le thème, le test de position et le focus, inchangés par rapport à tous les autres back-ends. |

L'hôte Java tient la place du serveur X sous Linux. Les pixels transitent par
un **memfd** que l'application crée et remet à l'hôte par le socket ;
le côté Go écrit du RGBA_8888, qui est octet pour octet l'ARGB_8888 d'Android, si bien que
la copie se fait sans conversion, et seulement pour le rectangle endommagé. Les entrées,
les marges (insets), le texte IME et l'arbre d'accessibilité passent par un protocole tramé sur un
`LocalSocket` abstrait.

```go
c, err := android.Dial("my app", nil) // nil theme: toolkit.DefaultDark()
if errors.Is(err, android.ErrUnsupported) {
	return nil // not under a host
}
defer c.Close()
return c.Run(myWidgetTree())
```

`Client` satisfait le `Backend` de `window`, si bien qu'une application passe d'Android
aux back-ends de bureau sans changer une ligne au-dessus de la fenêtre —
et [`window.Open`]({{< relref "/surfaces/native-window.md#how-open-chooses" >}})
le choisit de lui-même quand le socket de l'hôte est présent.

## arm64 uniquement {#arm64-only}

`android/arm64` est la seule cible Android que Go sait lier sans cgo ; `arm`,
`amd64` et `386` exigent une édition de liens externe. La CI vérifie les deux moitiés — celle
qui se compile et les trois qui ne se compilent pas — si bien que le jour où Go lèvera cette restriction,
la compilation le dira.

## Accessibilité {#accessibility}

Un lecteur d'écran verrait la `SurfaceView` comme un unique rectangle opaque. L'hôte
lui donne à la place une hiérarchie de vues virtuelle : un nœud par widget accessible,
avec la classe `android.widget.*` d'après laquelle Android choisit ses annonces, le
texte, les limites, et une action d'activation. L'arbre est **tiré** quand
quelque chose le lit, jamais poussé, si bien qu'une application à laquelle aucun service d'accessibilité
n'est attaché n'en construit jamais. Une activation revient sous forme d'un clic ordinaire au
centre de l'élément, par le même code que celui d'un toucher.

## Tactile et animation {#touch-and-animation}

Chaque échantillon tactile atteint l'arbre sous forme d'un événement tactile, puis d'un événement souris : le
premier pour les widgets qui comprennent les gestes, le second pour les nombreux qui n'écoutent que les
clics. Les widgets animés avancent sur une boucle d'images qui démarre quand quelque chose
commence à s'animer et s'arrête quand plus rien ne s'anime.
