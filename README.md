# myseq instruments

Instrumentos MSP para las cinco salidas de `myseq`. La primera implementación es `myseq.kick~`.

## `myseq.timecode~`

Generador de sincronía con interfaz gráfica. El outlet izquierdo produce señal LTC/SMPTE sample-accurate; el derecho entrega bytes MIDI Time Code crudos y se conecta directamente a `midiout`.

Soporta 24, 25, 29.97 drop-frame y 30 fps. MTC emite los ocho mensajes Quarter Frame y mensajes Full Time Code SysEx al arrancar, localizar, cambiar de velocidad o solicitar `fullframe`.

```text
myseq.timecode~
start
stop
time 1 23 45 12
locate 1 23 45 12
now
reset
fps 29.97df
level -18.
rise 40.
ltc 1
mtc 1
device 127
fullframe
dump
```

La salida LTC está destinada a una entrada de timecode, una pista de grabación o hardware de sincronía; no debe monitorizarse a volumen de escucha. Para generar LTC hay que activar el DSP de Max. El reloj MTC puede seguir funcionando aunque se silencie LTC.

## `megaseq`

`megaseq` conserva el motor completo de `myseq` y lo reorganiza como un instrumento radial. Tiene cinco anillos concéntricos y cinco outlets independientes, de izquierda a derecha:

1. kick, canal MIDI 1
2. snare, canal MIDI 2
3. hihat, canal MIDI 5
4. melody, canal MIDI 3
5. bass, canal MIDI 4

Cada outlet entrega listas compatibles con los instrumentos modulares:

```text
pitch velocity duration_ms channel
```

El secuenciador incluye los mismos sistemas de patrones, Euclid, spray, ratchets, escalas, motif, progression, Markov, Delaunay/Voronoi, fluid, pollution, composer, arpeggiator, automatización reversible, escenas, morph, locks y bancos persistentes de `myseq`.

La interacción también es radial:

- Los cinco anillos son kick, snare, hihat, melody y bass; se pueden pintar pasos directamente.
- El núcleo central edita root, scale y las doce pitch classes.
- Los ocho satélites exteriores son escenas; Alt-click almacena y click recupera.
- El navegador lateral organiza voz, sección, página musical, automatización, diagnósticos y banco en una jerarquía limpia y directa.
- El deck inferior organiza el banco activo en una sola fila de módulos verticales compactos, sin invadir el radial.
- El anillo de energía interior edita los 16 puntos de la curva de frase.

### Hybrid Rhythm Lab

La página `RHYTHM` mezcla las dos lecturas del secuenciador: mantiene las cinco órbitas radiales y abre un `Rhythm Ribbon` compacto debajo del círculo para pintar pasos con precisión, sin taparlo. Las celdas de color muestran la predicción del algoritmo; los marcadores blancos muestran el patrón manual. Ambos estratos se mezclan realmente en el disparo.

Cada voz puede usar un motor independiente:

- `original`: conserva el comportamiento clásico de `myseq`.
- `euclid`: distribuye pulsos uniformemente dentro del ciclo.
- `polymeter`: transforma la distribución usando un ciclo independiente por voz.
- `clave`: aplica esqueletos sincopados adaptados a kick, snare, hihat, melody y bass.
- `cellular`: genera ritmos mediante autómatas celulares evolutivos.
- `burst`: libera grupos de eventos cerca de los límites de frase.
- `hybrid`: combina votación Euclid/clave/cellular con acentos dinámicos.

Los controles `PULSE COUNT`, `POLYMETER CYCLE`, `ORBIT ROTATION`, `RHYTHM VARIATION`, `SYNCOPATION`, `GHOST NOTES` y `PATTERN EVOLUTION` son específicos para la voz seleccionada. Escenas, morph y `savebank` almacenan el estado completo de estos motores.

Uso básico y ruteo modular:

```text
megaseq
start
tempo 128
length 16
root 60
scale minor
automation profile flow
scene 1 store
rhythm kick hybrid 5 16 0 0.35 0.45 0.12 0.3
rhythm melody polymeter 5 9 1 0.25 0.5 0.08 0.4
rhythmregenerate all

outlet 1 -> myseq.kick~
outlet 2 -> myseq.snare~
outlet 3 -> myseq.hihat~
outlet 4 -> myseq.melody~
outlet 5 -> myseq.bass~
```

## `myseq.kick~`

Recibe directamente el evento producido por la salida de kick del secuenciador:

```text
pitch velocity duration_ms channel
```

Ejemplo:

```text
36 116 260 1
```

El instrumento tiene una interfaz gráfica redimensionable, ocho voces, síntesis FM transitoria, envolvente tonal, click de ruido filtrado, saturación y sobremuestreo 2×. `clear` o `panic` apagan todas las voces. `bang` dispara el kick predeterminado.

Los gates cortos producidos por el secuenciador no recortan la cola del bombo: `decay` establece su duración mínima y `duration_ms` puede extenderla.

Parámetros principales:

```text
decay 260
tune 0
pitchdrop 36
pitchdecay 24
fmratio 1.5
fmindex 2.4
fmdecay 18
click 0.18
clickdecay 4
drive 0.32
level 0.85
channel 1
omni 0
```

También se pueden definir al crear el objeto:

```text
myseq.kick~ @decay 320 @fmindex 3.2 @drive 0.45
```

## Interfaz y mutación

La GUI incluye:

- Osciloscopio y medidor de salida en tiempo real.
- Plano XY entre cuatro identidades: `DEEP`, `PUNCH`, `CLICK` y `METAL`.
- Trayectoria visible de la mutación automática.
- Diez controles editables de cuerpo, pitch, FM, click, drive y nivel.
- Marcadores separados para el valor manual y el valor efectivo después del morph.
- Controles de cantidad, velocidad y mezcla de morph.
- Botones de audición, mutación, congelación y randomización.

Arrastra horizontalmente los controles. Arrastra dentro del campo XY para cambiar de carácter. Un doble click sobre un control restaura su valor predeterminado.

Mensajes de performance:

```text
character deep
character punch
character click
character metal
character random
morph 0.25 0.70 1.
mutate 1 0.45 0.18
mutation 0.65 0.30
freeze 1
randomize
seed 12345
dump
```

Después de recompilar una versión ya cargada, reinicia Max para que descargue de memoria el `.mxo` anterior y cargue la GUI nueva.

El encabezado muestra `DSP ON` cuando el instrumento forma parte de una cadena de audio activa. Si muestra `DSP OFF`, activa `ezdac~` o revisa la conexión de señal. Los eventos recibidos con DSP apagado no se acumulan en la cola.

## Pruebas del núcleo DSP

No requieren el Max SDK:

```sh
cmake -S . -B build-test -DMYSEQ_BUILD_EXTERNALS=OFF
cmake --build build-test
ctest --test-dir build-test --output-on-failure
```

## Compilación del external

CMake detecta automáticamente las instalaciones habituales, incluida `/Applications/max-sdk-main`. Si no encuentra una copia local, descarga `max-sdk-base` de Cycling '74. También se puede indicar una ruta explícita:

```sh
cmake -S . -B build -DMAX_SDK_BASE_DIR=/ruta/a/max-sdk-base
cmake --build build --config Release
```

En macOS el resultado se genera en `externals/myseq.kick~.mxo` como binario universal Intel/Apple Silicon.

## `myseq.snare~`

Recibe directamente la segunda salida de `myseq` (`pitch velocity duration_ms channel`, canal 2). Combina dos modos resonantes, ruido filtrado, transient snap, rattle, FM y saturación con ocho voces y sobremuestreo 2×.

La GUI interpola entre cuatro identidades:

- `TIGHT`: corto, preciso y con ataque marcado.
- `WOOD`: cuerpo modal grave y menos ruido.
- `AIR`: cola de ruido amplia y rattle móvil.
- `METAL`: relaciones inarmónicas y FM intensa.

Además de la mutación continua, `VARIATION` genera pequeñas diferencias deterministas en afinación modal, color y rattle en cada golpe.

```text
character tight
character wood
character air
character metal
character random
morph 0.4 0.7 1.
mutate 1 0.5 0.16
mutation 0.75 0.3
freeze 1
randomize
seed 24680
```

El objeto no necesita argumentos:

```text
myseq.snare~
```

Después de recompilar, reinicia Max y conecta la segunda salida de `myseq` a `myseq.snare~`.

## `myseq.melody~`

Sintetizador melódico estéreo para la cuarta salida de `myseq` (canal MIDI 3). Es polifónico a 12 voces y combina un núcleo FM/wavefold con hasta 128 granos, dispersión afinada, paneo por grano y delay cruzado estéreo. El procesamiento interno usa sobremuestreo 2×.

Las cuatro identidades del campo XY son:

- `GLASS`: granos cristalinos, consonantes y brillantes.
- `BLOOM`: ataques suaves, colas largas y un centro tonal cálido.
- `DUST`: nube granular corta, dispersa y puntillista.
- `ORBIT`: FM, fold, drift y feedback más radicales.

`HARMONICITY` interpola entre intervalos consonantes y fracturados, por lo que es posible llevar el instrumento hacia territorio avant-garde sin perder completamente el ancla musical. Cada nota captura el estado efectivo del morph y puede coexistir con notas anteriores.

```text
character glass
character bloom
character dust
character orbit
morph 0.35 0.7 1.
mutate 1 0.55 0.11
mutation 0.7 0.22
freeze 1
randomize
panic
```

No necesita argumentos:

```text
myseq.melody~
```

También admite atributos al crearlo, por ejemplo:

```text
myseq.melody~ @density 36 @grainsize 120 @harmonicity 0.9 @feedback 0.3
```

Conecta la salida 4 de `myseq` a su entrada y sus dos salidas de señal a los canales izquierdo y derecho. Activa `ezdac~` antes de iniciar el secuenciador.

## `myseq.mega~`

Sintetizador estéreo polifónico de 16 voces para las listas MIDI de `myseq`. Cada voz combina una red FM de seis operadores y cuatro topologías con un megaoscilador de seno, triángulo, sub y masa unísona. Incluye wavefold, ring modulation, caos determinista, filtro resonante, transitorios, dimensión estéreo y cuatro procesos de glitch de corta duración. Todo el núcleo no lineal trabaja con sobremuestreo 2×.

`INTELLIGENCE` acerca las relaciones de los operadores y la mutación hacia proporciones armónicas estables. Al bajarlo, `RATIO SPREAD`, `VOICE MUTATION`, `CHAOS` y `GLITCH` pueden producir estructuras más fracturadas sin cambiar el pitch MIDI recibido.

Organismos del campo XY:

- `GRID`: articulación precisa, relaciones consonantes y glitches rítmicos discretos.
- `SWARM`: megaoscilador, unison, detune y dimensión estéreo.
- `CRYSTAL`: FM brillante, wavefold y resonancia musical.
- `FRACTURE`: topologías complejas, ring modulation, caos y glitch agresivo.

```text
myseq.mega~
character grid
character swarm
character crystal
character fracture
morph 0.45 0.7 0.9
mutate 1 0.55 0.14
evolve
preset 1 store
preset 1 recall
panic
```

Recibe `pitch velocity duration_ms channel`; por defecto escucha el canal 3. La GUI incluye 20 controles, osciloscopio estéreo, contador de voces/glitches, presets persistentes, tooltips y feedback visual de todos los botones.

## `myseq.hihat~`

Hi-hat estéreo mutable para la tercera salida de `myseq` (canal MIDI 5). Combina seis osciladores metálicos inarmónicos, ring modulation, FM, ruido con high-pass dinámico, strike transitorio, sizzle, saturación y sobremuestreo 2×.

El parámetro `OPENNESS` interpola entre las envolventes `CLOSED DECAY` y `OPEN DECAY`. Un golpe cerrado estrangula automáticamente las colas abiertas anteriores; una duración MIDI larga también puede abrir la articulación.

Identidades del campo XY:

- `TIGHT`: cerrado, brillante, preciso y centrado.
- `SILK`: aire suave, ruido amplio y estéreo.
- `TRASH`: metal áspero, oscuro y saturado.
- `ALIEN`: FM intensa, afinación extraña y movimiento espacial.

```text
character tight
character silk
character trash
character alien
openness 0.75
morph 0.35 0.7 1.
mutate 1 0.55 0.18
mutation 0.7 0.3
freeze 1
randomize
panic
```

No requiere argumentos:

```text
myseq.hihat~
```

También puede configurarse al crearlo:

```text
myseq.hihat~ @openness 0.2 @brightness 0.85 @inharmonicity 0.9 @spread 0.7
```

Conecta la salida 3 de `myseq` a la entrada de `myseq.hihat~` y las dos salidas de señal a izquierda/derecha.

## Presets de usuario persistentes

Los cinco instrumentos, `myseq.fx~` y `myseq.sprayverb~` tienen 16 slots propios. Guardan todos los parámetros manuales, el campo de performance/morph y su configuración de movimiento o mutación. Los bancos predeterminados viven en `~/.myseq/presets/` y se cargan automáticamente al crear el objeto.

```text
preset 1 store MiSonido
preset 1 recall
preset 1 clear
preset list
preset save
preset load
```

`preset store` y `preset clear` escriben inmediatamente el banco predeterminado. Para exportar o importar otro banco se puede añadir una ruta:

```text
preset save /ruta/mis_bajos.bank
preset load /ruta/mis_bajos.bank
```

Este protocolo funciona igual en `myseq.kick~`, `myseq.snare~`, `myseq.hihat~`, `myseq.melody~`, `myseq.bass~`, `myseq.fx~` y `myseq.sprayverb~`.

## Tooltips contextuales

Las siete interfaces muestran ayuda después de mantener el puntero aproximadamente 260 ms sobre un elemento. Los tooltips incluyen el valor efectivo y explican controles de síntesis/procesamiento, campo XY, caracteres, botones de performance, los 16 slots de preset y los macrocontroles inferiores.

Están activos de forma predeterminada. Se pueden desactivar desde el Inspector o al crear cualquier instrumento:

```text
myseq.kick~ @tooltips 0
myseq.snare~ @tooltips 0
myseq.hihat~ @tooltips 0
myseq.melody~ @tooltips 0
myseq.bass~ @tooltips 0
myseq.fx~ @tooltips 0
myseq.sprayverb~ @tooltips 0
```

## `myseq.fx~`

Procesador multiefectos estéreo para performance en vivo. Se inserta después de cualquiera de los instrumentos y mantiene un buffer circular de ocho segundos para stutter, freeze y capturas glitch. También incluye reverse continuo, jitter de velocidad, feedback, bit crush, decimación, ring modulation, filtro, expansión estéreo, drive y dry/wet.

Tiene dos entradas y dos salidas de señal. Si la fuente es mono, conecta únicamente la entrada izquierda: el external la copia automáticamente a ambos canales.

Acciones de performance:

```text
stutter 1
stutter 0
freeze 1
freeze 0
glitch
bypass 1
motion 1
xy 0.82 0.68 0.75
character pulse
character ice
character shred
character void
randomize
clear
```

El campo XY mezcla los parámetros manuales con un macro de fragmentación/destrucción; `MOTION` recorre ese campo automáticamente. Los botones `STUTTER` y `FREEZE` quedan activados hasta volver a pulsarlos, mientras `GLITCH` es momentáneo.

`myseq.fx~` también tiene 16 presets persistentes independientes, tooltips contextuales y los mismos botones `STORE`, `SAVE` y `LOAD` de los instrumentos.

## `myseq.sprayverb~`

Reverb granular estéreo con un visualizador de partículas alimentado por los eventos reales de emisión de granos. Combina hasta 96 granos, historial estéreo, shimmer, modulación y una red FDN de ocho líneas.

Su sistema `BREATH` crea espacios atmosféricos generativos: reduce la densidad y la intensidad de la nube, deja respirar el material seco y después libera automáticamente un spray granular. `SPACE` provoca manualmente un ciclo de vacío y liberación; `BURST` suelta la nube inmediatamente.

```text
space
burst
breath 1
freeze 1
freeze 0
motion 1
xy 0.75 0.82 0.70
character mist
character halo
character shimmer
character storm
randomize
clear
```

Tiene entrada y salida estéreo, copia mono automática desde la entrada izquierda, 16 presets persistentes, tooltips y cuatro atmósferas base. `FREEZE` detiene la entrada y mantiene la red de difusión cerca de una cola infinita.

Cada GUI incluye además dos filas de ocho slots. Haz click en un slot para recuperarlo; pulsa `STORE` y después un slot para guardarlo. Alt-click sobre un slot guarda directamente. `SAVE` escribe el banco persistente y `LOAD` vuelve a leerlo desde disco.

## `myseq.bass~`

Bajo estéreo polifónico para la quinta salida de `myseq` (canal MIDI 4). Combina subgrave mono-compatible, cuerpo triangle/saw estéreo, síntesis FM, wavefolding, filtro resonante de cuatro etapas, envolvente de filtro, saturación y glide.

Identidades del campo XY:

- `SUB`: grave limpio, centrado y profundo.
- `FUNK`: ataque corto, filtro dinámico y cuerpo percusivo.
- `REESE`: ancho, detuned, plegado y agresivo.
- `ACID`: resonancia, modulación de filtro y glide pronunciado.

```text
character sub
character funk
character reese
character acid
morph 0.3 0.75 1.
mutate 1 0.45 0.09
preset 1 store MiBajo
preset 1 recall
```

Conecta la salida 5 de `myseq` a `myseq.bass~`; sus dos salidas de señal son izquierda y derecha.
