---
title: "El toolkit"
linkTitle: "Toolkit"
weight: 20
description: "160 tipos de widget dibujados a través de la costura del painter: temas, layouts, texto, entrada, accesibilidad."
tags: [renderizado, toolkit, widgets, tema, accesibilidad]
---

[`go-widgets/toolkit`](https://github.com/go-widgets/toolkit) es el conjunto de
widgets. Todos los widgets implementan la misma interfaz —`Bounds`, `SetBounds`,
`Draw(p painter.Painter, theme *Theme)`, `HitTest`, `OnEvent`—, de modo que un
host trata igual un botón, una tabla y un calendario de eventos. Solo la
biblioteca estándar, `CGO_ENABLED=0`, 100 % de cobertura de sentencias.

En la v0.329.0 tiene **160 tipos de widget exportados** en unas **69 000 líneas**
de código de widgets. Esas dos cifras se midieron en esa etiqueta, no se
arrastraron de antes: el README del toolkit da los comandos, y conviene volver
a ejecutarlos en lugar de repetir las cifras.

## Las familias {#the-families}

| Familia | Widgets |
|---|---|
| Acción | `Button`, `ToggleButton`, `CheckButton`, `RadioButton` + `RadioGroup`, `Switch`, `SplitButton`, `IconButton`, `CycleButton`, `Chip`, `SegmentedBar` |
| Entrada | `Entry`, `TextView` (selección, vista previa de IME, tramos de sintaxis, números de línea), `SpinButton`, `Scale`, `RangeSlider`, `SearchEntry`, `TagField`, `ComboBox`, `FormField` con `Validate`/`Rule` |
| Selección | `ListBox`, `TreeView`, `DropDown` |
| Contenedores | `Container` con un `Layout` intercambiable (`FitLayout`, `BoxLayout`, `BorderLayout`, `CardLayout`, `FlowLayout`); `HBox`, `VBox`, `Grid`, `Frame`, `Dock`, `Border`, `Stack`, `Overlay`, `Paned`, `Splitter`, `Expander`, `Accordion`; un constructor declarativo `Node` |
| Navegación | `Menu`, `MenuBar` (con `Alt`+letra), `ContextMenu`, `Popover`, `CommandPalette`, `Dialog`, `MessageDialog`, `Wizard`, `Notebook`, `ViewSwitcher`, `Carousel` |
| Respuesta | `ProgressBar`, `ProgressCircle`, `LevelBar`, `Spinner`, `Tooltip`, `Notification`, `Toast`, `Banner`, `Alert`, `Badge`, `Skeleton` |
| Datos | `Table` (edición de celdas, columnas fijas, filas de grupo, agregados, expansores de fila), `TreeTable`, `PropertyGrid`, `Kanban`, `Gantt`, `Agenda` (semana, mes, trimestre, año; varios calendarios) |
| Gráficos | `LineChart`, `BarChart`, `PieChart`, `AreaChart`, `ScatterChart`, `RadarChart`, `Gauge`, `Sparkline` |
| Compuestos | `FileChooser`, `ColorChooser`, `ColorPicker`, `FontChooser`, `Calendar`, `DatePicker`, `DateRangePicker`, `TimePicker`, `MarkdownView`, `MarkdownEditor`, `TerminalView` |
| Shell | `Window` con decoraciones del lado del cliente, `HeaderBar`, `Toolbar`, `Ribbon`, `Statusbar`, `StatusIcon`, `StatusArea`, `Wallpaper`, `Thumbnail` |

Todos ellos, en vivo: la [galería](https://go-widgets.github.io/gallery/).

## Cinta {#ribbon}

`Ribbon` es una cinta de comandos al estilo de Office: pestañas de grupos
rotulados con botones grandes (icono sobre el texto), pequeños (icono al lado),
interruptores, botones divididos, listas desplegables y una pequeña galería.
Sigue las pautas de las cintas de Windows: al estrecharse la ventana, cada
grupo se reduce según su prioridad — completo, compacto, solo iconos y por
último un único botón de menú —, de modo que la cinta muestra la mayor
combinación que cabe; se contrae a sus pestañas (Ctrl+F1 o el chevrón) y una
pestaña deja entonces ver su contenido; Alt muestra las pistas de teclado; las
descripciones emergentes dan el atajo y una descripción. La navegación por
pestañas y por barra de herramientas sigue los patrones WAI-ARIA, y sus
contornos y su anillo de foco mantienen un contraste de 3:1 en temas claros y
oscuros.

Cada control es una vista de una `Action` — el comando único que comparten un
menú, una paleta y la cinta — y `NewCommandAction` crea una a partir de un
`mvvm.Command` del modelo de vista, cuyo `CanExecute` atenúa el control. La
pestaña activa, el estado contraído, un interruptor y la elección de una lista
son `mvvm.Observable`. Bajo un `PopoverHost`, sus menús se dibujan sobre la
página sin código del anfitrión.

`RibbonGrid` es una clase de control cuyo menú emergente es una cuadrícula de
`GridCols` por `GridRows` celdas (8 por 6 si es cero, 16 por 16 como máximo),
como el botón Tabla de un procesador de texto. El puntero o las flechas
iluminan las celdas desde la esquina hasta la que queda debajo y una leyenda
dice «3 × 2»; un clic o Intro elige y llama a `OnGrid(cols, rows)`; Inicio y
Fin saltan, Esc cierra. Se dibuja en claro y oscuro y se comunica a las
tecnologías de apoyo como un botón expandido mientras la cuadrícula está
abierta.

```go
&toolkit.RibbonControl{Kind: toolkit.RibbonGrid, Label: "Table",
	GridCols: 8, GridRows: 8,
	OnGrid: insertTable}
```

## Divisor {#splitter}

`Splitter` dispone N paneles uno junto a otro (o uno sobre otro con `vertical`
en verdadero) y coloca entre cada par vecino un divisor arrastrable y con foco,
según el patrón WAI-ARIA «window splitter». Un panel es **fijo** — su `Size`, en
píxeles lógicos, es un `mvvm.Observable[int]` que escribe el divisor — o
**flexible**, y se reparte lo que dejan los paneles fijos según su `Weight`,
también un `Observable`. Cada panel tiene un `Min` y un `Max`. Un panel fijo
`Collapsible` se cierra (tamaño 0) al arrastrar el divisor por debajo de la
mitad de su `Min`, o con doble clic o Intro, y se vuelve a abrir igual. El
observable `Visible` de un panel lo apaga junto con su divisor. Como los tamaños
viven en observables, un modelo de vista puede enlazarlos, guardarlos y
restaurarlos; `OnResized` se llama una vez por gesto terminado con los paneles
que la persona cambió, nunca por una disposición que solo siguió el cambio de
tamaño de la ventana.

```go
files := mvvm.NewObservable(240) // fixed pane, logical px
w := mvvm.NewObservable(1)       // flexible pane, weight

s := toolkit.NewSplitter(false,
	&toolkit.SplitPane{Widget: tree, Name: "Files", Size: files,
		Min: 120, Max: 480, Collapsible: true},
	&toolkit.SplitPane{Widget: editor, Name: "Editor", Weight: w, Min: 200},
)
s.OnResized = func(changed []int) { save(files.Get(), w.Get()) }
```

Un divisor recibe el foco: las flechas (arriba y abajo en vertical) lo mueven
`Step` píxeles lógicos, cuatro veces más con Mayús; Inicio y Fin llevan el panel
que lo precede a su tamaño mínimo y máximo; Intro cierra o restablece el panel
fijo que controla; Tab pasa de un divisor a otro. Cada divisor es un
`RoleSeparator` con `aria-valuemin`, `aria-valuemax` y `aria-valuenow`, llamado
«Resize» más el `Name` del panel. `Cursor(x, y)` devuelve `"col-resize"` o
`"row-resize"` sobre un divisor, y `SplitterColors(theme)` da la línea y el
acento del divisor, que mantienen un contraste de 3:1 frente al fondo y la
superficie en ambos temas por defecto, igual que el anillo de foco.

## Temas {#themes}

Un único valor `Theme` se propaga en cascada por todo el árbol: cambie un color
en la raíz y todos los widgets se repintan con él. `DefaultLight()` y
`DefaultDark()` vienen incorporados, y `LoadGTKTheme(css)` lee el bloque
`@define-color` de cualquier tema de GTK 3 o libadwaita —Adwaita, WhiteSur,
Solarized— y lo convierte en un `Theme`.

## Texto {#text}

La fuente compilada por defecto es una fuente de mapa de bits de 5×7, para que
las pruebas de píxeles conserven su geometría. Una sola llamada pasa toda la
interfaz a texto con antialiasing y conformado:

```go
toolkit.UseOpenTypeText() // Atkinson Hyperlegible, bundled; js/wasm-safe
```

La tipografía procede de [go-opentype/fonts](https://github.com/go-opentype/fonts)
y la rasteriza [go-opentype](https://github.com/go-opentype/opentype), en Go,
sin biblioteca de fuentes en C ni búsqueda de fuentes del sistema.
`NewTrueTypeFont(ttf, px)` carga otra tipografía; `NewFallbackFont` encadena
varias, por ejemplo para añadir una tipografía CJK o árabe; el campo `Font`
propio de un widget sustituye la global solo para ese widget.

## Entrada {#input}

Ratón, teclado, composición IME, táctil (`EventTouchStart`/`Move`/`End`, con
`GestureRecognizer` para toques, pulsaciones largas y deslizamientos),
arrastrar y soltar (`DragSource`, `DropTarget`). La selección de varias filas en
`ListBox`, `Table` y `TreeView` funciona con el ratón y con el teclado, y
respeta ⌘ además de Ctrl, porque en macOS Ctrl-clic es el clic secundario.

## Tamaño del binario: un programa enlaza lo que usa {#binary-size}

El toolkit es un paquete grande, y para el enlazador eso no es un problema: la
eliminación de código muerto de Go solo conserva lo alcanzable desde
`main.main` **y desde las tareas de inicialización de cada paquete
importado**. Los widgets que un programa nunca toca se descartan. Lo que no se
descarta es todo lo que alcanza un inicializador a nivel de paquete o el `init`
de un paquete importado; por eso, desde la v0.345.0, el toolkit mantiene sus
propios inicializadores sin trabajo y evita importar paquetes cuyo `init`
arrastra código:

- `Material` y `Thumbnail` usan
  [`github.com/go-images/images/ops`](https://github.com/go-images/images), sin
  códecs, y no `github.com/go-images/images`, cuyos decodificadores se
  registran en `init` (unos 1,7 MB de código WebAssembly).
- `CollabText`, `NewCollabText`, `MetaName` y `MetaColor` pasaron a
  **`github.com/go-widgets/toolkit/collabui`**: necesitan una sesión
  `go-crdt/collab` viva, cuyos transportes traen `net/http` y `crypto/tls`.
  `CollabSheet` e `IsoCRDTDocument` solo necesitan `go-crdt/crdt` y se quedan
  en `toolkit`.
- La expresión regular de la `Rule` de correo, el registro de iconos iso
  integrado y la tabla de botones de `RichEditorToolbar` se construyen la
  primera vez que se usan, no en inicializadores.

⛔ **El traslado es un cambio incompatible, sin redirección obsoleta.** Cambie
el import de los cuatro nombres anteriores a `collabui`; nada más cambia.

Tamaños para `GOOS=js GOARCH=wasm` con `-ldflags="-s -w"`, en bruto y gzip -9:

| Programa | antes (bruto / gzip) | después (bruto / gzip) |
|---|---|---|
| un `Label` (v0.345.0) | 10,49 / 2,88 MB | 7,48 / 2,13 MB |
| un `Label` (v0.345.1) | 10,49 / 2,88 MB | 5,66 / 1,61 MB |
| go-wasmcontainers/dashboard | 20,42 / 5,17 MB | 17,72 / 4,58 MB |
| el mapa de flujo del relay | 22,62 / 5,44 MB | 19,95 / 4,85 MB |

Lo que queda es sobre todo el runtime y la pila OpenType que necesita el
renderizado de texto. Para ver qué retiene un símbolo en su programa, compile
con `-ldflags=-dumpdep` y siga la cadena de primeros padres desde el símbolo
hasta `main.main` o `go:main.inittasks`.

La v0.345.0 añade también `ColorHex` y `ParseColorHex`, para escribir un color
en hexadecimal y volver a leerlo.

## Accesibilidad {#accessibility}

Un toolkit que pinta sus propios píxeles no da a un lector de pantalla nada que
leer a menos que diga lo que ha dibujado. Los widgets implementan `Accessible`
—un rol, un nombre, un valor— y `CollectA11y` los reúne para que el host los
publique: AT-SPI en Linux a través de
[`window`]({{< relref "/surfaces/native-window.md" >}}), y una jerarquía de
vistas virtual en [Android]({{< relref "/surfaces/android.md#accessibility" >}}).
