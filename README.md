# myseq instruments

Max/MSP instruments for the five outputs of `myseq`, plus generative sequencers, live effects, granular reverb, real-time vocal looping, experimental drum synthesis, and timecode synchronization.

## `myseq.polydrum~`

`myseq.polydrum~` is a procedural drum synthesizer and shared-pulse groove composer. It now has its own synthesis engine instead of reusing the granular drum core: kick, snare, hat, clap, tom, rim, shaker, and fx are rendered as dedicated FM/physical/noise drum voices with bounded variation.

The eight voices are `kick`, `snare`, `hat`, `clap`, `tom`, `rim`, `shaker`, and `fx`. Each voice owns `cycle`, `pulses`, `rotation`, `probability`, `repeat`, `nudge`, `tune`, and `gain`. Global macros shape the complete system: `groove`, `euclid`, `repeat`, `fill`, `body`, `transient`, `variation`, and `room`.

It can run from its own clock or react to standard `myseq` / `megaseq` events:

```text
pitch velocity duration_ms channel
```

Common messages:

```text
start
stop
reset
variant
tempo 137.
param groove 0.85
param euclid 0.28
param repeat 0.22
param fill 0.30
param body 0.46
param transient 0.64
poly kick 16 4 0 0.95
poly hat 16 8 0 0.82
poly snare 16 2 12 0.88
voice shaker nudge 0.56
voice fx tune 0.58
trigger rim 1.
env kick fm 2 0.06 0.92
env kick pitch 3 0.24 0.18
env snare noise 3 0.18 0.55
envreset kick fm
preset 1 store DeepGrid
preset 1 recall
savebank
loadbank
```

The GUI shows a shared 16-step groove grid: each lane displays its voice pattern, current cursor, pulse/cycle relationship, and meter. Click a lane to select/audition it, then edit its rhythm and synthesis behaviour from the lower controls.

Each voice also has an editable envelope graph with four tabs: `AMP`, `PITCH`, `FM`, and `NOISE`. These are not cosmetic curves: `AMP` shapes final level, `PITCH` controls pitch drop/rise, `FM` controls modulation depth and metallicity, and `NOISE` controls transient/noise energy. Drag the five breakpoints in the GUI or send `env <voice> <amp|pitch|fm|noise> <point 1-5> <time 0-1> <value 0-1>`. Presets are stored at `~/Documents/Max 9/Library/myseq.polydrum.bank.txt` and include all envelope graphs.

## `myseq.grainmachine~`

`myseq.grainmachine~` is an eight-voice granular drum machine with an internal 32-step sequencer. Each drum hit is generated as a short synthetic micro-source and then reconstructed as a cloud of grains. The voices are `kick`, `snare`, `hat`, `clap`, `tom`, `rim`, `shaker`, and `fx`.

The synthesis engine combines procedural drum bodies, noise excitation, metallic partial banks, reverse grains, inverse/rising envelopes, temporal spray, resynthesis weight, per-voice chaos, bus drive, and a small cross-feedback spatial smear. It can run from its own clock or respond to the standard `myseq` / `megaseq` event list:

```text
pitch velocity duration_ms channel
```

Channels 1, 2, and 5 map naturally to kick, snare, and hat. The GUI provides an editable 8×32 step grid, global grain macros, selected-voice sound design controls, live grain-field display, tooltips, and sixteen persistent presets. The preset bank is stored at `~/Documents/Max 9/Library/myseq.grainmachine.bank.txt`.

Common messages:

```text
start
stop
reset
default
random
tempo 132.
length 16
param density 0.86
param graincount 0.72
param reverse 0.35
param inverse 0.50
param resynth 0.85
trigger kick 1.
step kick 13 0.8
voice snare texture 0.9
pattern hat 1 0 0.7 0 1 0 0.55 0
clearpattern all
preset 1 store BrokenDust
preset 1 recall
savebank
loadbank
```

## `myseq.voicelooper~`

`myseq.voicelooper~` is a stereo real-time vocal sampler, live looper, and granular performance instrument. Feed it a microphone, synth, field recording, or any Max signal; press `REC` to capture a circular loop, `OVERDUB` to layer material, and `FREEZE` to hold the current memory as a playable granular cloud.

The object accepts the same event list used by `myseq` and `megaseq`:

```text
pitch velocity duration_ms channel
```

Incoming events trigger pitch-aware grain bursts, so a vocal loop can follow melody or bass patterns while still behaving like an audio effect. The GUI includes a waveform monitor, granular spray field, five modes (`clean`, `cloud`, `choir`, `glitch`, `void`), mutation depth/rate controls, contextual tooltips, and twelve persistent preset slots. The preset bank is stored at `~/Documents/Max 9/Library/myseq.voicelooper.bank.txt`.

Common messages:

```text
record 1
record 0
overdub 1
freeze 1
freeze 0
clear
random
mode cloud
mode choir
mode glitch
length 4.
pitch 7.
param density 0.72
param spray 0.85
trigger 72 118 420 3
preset 1 store
preset 1 recall
savebank
loadbank
dump
```

## `myseq.recorder~`

`myseq.recorder~` is a real-time-safe stereo studio recorder with two signal inputs and two independent monitor outputs. Audio capture is transferred from the DSP thread through a lock-free queue to a dedicated disk writer, preventing file-system stalls from blocking the audio callback. It writes long-session RF64/WAV takes in dithered PCM 16-bit, dithered PCM 24-bit, or IEEE float32.

The GUI provides RECORD, PAUSE/RESUME, STOP and MARK transport controls, stereo peak/RMS meters, scrolling input history, elapsed time, file size, disk-drop diagnostics, clean input gain, independent monitor gain, up to ten seconds of pre-roll, optional transparent monitoring, and an optional soft safety limiter. Unconnected right input automatically records the left input as mono-compatible stereo.

Automatic takes are timestamped in `~/Music/Myseq Recordings/`. `path /custom/take.wav` sets the next destination. Markers are written with time and sample position to a companion `.markers.txt` file. Eight GUI presets and persistent SAVE/LOAD store recording format, gain, pre-roll, monitor, and limiter settings in `~/Documents/Max 9/Library/myseq.recorder.bank.txt`.

Messages include `record 1`, `record /path/take.wav`, `pause 1`, `pause 0`, `stop`, `format pcm24`, `format float32`, `inputgain -3.`, `monitorgain -9.`, `preroll 5.`, `monitor 1`, `limiter 1`, `marker CHORUS`, `preset 1 store`, `savebank`, `loadbank`, and `dump`.

## `myseq.midrouter`

`myseq.midrouter` is a dual-destination MIDI scale translator for connecting `myseq` or `megaseq` to differently tuned modular voices and Max instruments. It accepts the native event format `pitch velocity duration_ms channel` as well as three-item note lists and pitch integers. The left outlet is MODULAR and the right outlet is MAX.

Each destination has independent root, scale, chromatic transpose, octave, MIDI channel, velocity gain, duration scaling, enable state, mapping mode, and output format. `CHROMATIC` preserves semitone intervals, `QUANTIZE` shifts and snaps to a destination scale, and `DIATONIC` translates scale degrees between different source and target scales. Output can remain `EVENT 4`, become `NOTE 3` (`pitch velocity channel`), or use a legacy pitch integer.

The interactive GUI includes live note translation monitoring, route modes (`MODULAR`, `MAX`, `BOTH`, or `OFF`), eight user preset slots, persistent bank save/load, panic, and delayed tooltips. The default bank is stored at `~/Documents/Max 9/Library/myseq.midrouter.bank.txt`.

Core messages include `route both`, `source root 60`, `source scale minor`, `root modular 62`, `scale modular dorian`, `mode modular diatonic`, `transpose max -12`, `channel modular 1`, `format max event`, `preset 1 store`, `savebank`, and `loadbank`.

## `myseq.hybridmixer~`

A sixteen-input hybrid console with a stereo master. Connect the first eight outlets of `adc~ 1 2 3 4 5 6 7 8` to mixer inlets 1–8; `adc~` always follows the audio interface selected globally in Max under **Options → Audio Status**. Mixer inlets 9–16 accept instruments, effects, returns, or any other internal MSP signals.

Every channel includes input trim, high-pass and low-pass filters, drive, gate threshold, equal-power panorama, a -∞ to +12 dB fader, mute, additive solo, polarity inversion, peak/RMS metering, and four processing modes: `CLEAN`, `WARM`, `GATE`, and `EXCITE`. The stereo master provides dual peak/RMS meters, click-safe gain, mute, and a safety limiter.

```text
fader 1 -6.
pan 9 -0.35
trim 2 12.
hpf 1 80.
lpf 9 12000.
drive 9 0.6
gate 3 -48.
mode 9 warm
mute 4 1
solo 9 1
phase 2 1
label 9 MELODY
master -3.
mastermute 0
limiter 1
reset
dump
```

Sixteen persistent snapshots store the complete channel and master configuration:

```text
preset 1 store HybridSet
preset 1 recall
preset save
preset load
```

## Amoeba Outputs

`Amoeba Outputs` is a family of ten speaker-aware spatial processors. Each object accepts one audio signal and creates between 1 and 18 discrete MSP signal outlets. Choose the physical outlet count when the object is created:

```text
amoeba.outputs.orbit~ @outputs 8
amoeba.outputs.swarm~ @outputs 12
amoeba.outputs.granular~ @outputs 18
```

The outlet count is structural and therefore requires object recreation when changed. Every interface contains a radial speaker editor: drag numbered speaker nodes to match the physical room, use the selected-speaker azimuth/distance/elevation controls for precise placement, and drag the central amoeba core to offset the complete trajectory. `CIRCLE`, `FRONT`, `DOME`, and `RANDOM` create starting layouts.

The spatializer uses normalized equal-power distance panning against the edited speaker coordinates rather than assuming evenly spaced speakers. The ten organic motion systems are:

- `amoeba.outputs.orbit~`: eccentric orbits, precession, comet paths, and gravity wells.
- `amoeba.outputs.swarm~`: flocking agents with cohesion, separation, panic, and murmuration.
- `amoeba.outputs.flow~`: curl-field advection, tides, rapids, and turbulent streams.
- `amoeba.outputs.growth~`: branching tips, phyllotaxis, fern, coral, and bloom structures.
- `amoeba.outputs.physarum~`: nutrient-seeking slime agents and connected vein behavior.
- `amoeba.outputs.membrane~`: elastic-body waves, surface tension, stretch, and rupture.
- `amoeba.outputs.vortex~`: paired counter-rotating vortices and spiral attractors.
- `amoeba.outputs.brownian~`: inertial random walks from dust through plasma motion.
- `amoeba.outputs.granular~`: four-second multihead time grains distributed as a moving cloud.
- `amoeba.outputs.constellation~`: speaker-aware Lissajous nodes, webs, and spatial galaxies.

All objects also accept the standard sequencer event list. Events inject velocity- and pitch-dependent energy into the active spatial behavior:

```text
pitch velocity duration_ms channel
```

### Speaker geometry and performance messages

```text
speaker 1 -45. 1.0 0.
speaker 2 45. 0.85 25.
layout circle
layout front
layout dome
layout random
center 0.2 -0.15
auto 1
freeze 1
bypass 1
character random
param 1 0.5
set turbulence 0.72
randomize
reset
dump
```

`speaker` arguments are `one_based_output azimuth_degrees distance elevation_degrees`. Each module and outlet count owns an independent 16-slot persistent preset bank. Presets retain all twelve behavior controls, the core position, automation state, and coordinates for all eighteen possible speakers.

```text
preset 1 store MyRoom
preset 1 recall
preset save
preset load
```

## Massive module expansion

Ten additional MSP modules extend `megaseq` into physical modeling, rewriting systems, cellular growth, modulation, buffering, filtering, delay, resonance, and biological dynamics. Every module uses the same resizable interactive shell: stereo scope, four-corner morph field, twelve algorithm-specific controls, mutation, automatic motion, freeze, randomization, tooltips, and 16 independent persistent user presets.

All ten objects accept the standard `megaseq` event format:

```text
pitch velocity duration_ms channel
```

The event instruments generate sound directly. Audio processors use incoming events as additional excitation or modulation. Every object also has two signal inlets and two signal outlets; a mono signal connected to inlet 1 is copied automatically when needed.

### Event instruments

- `myseq.physpluck~`: 12-voice Karplus–Strong string field with stiffness, dispersion, body resonance, nonlinear tension, sympathetic input coupling, and per-note material variation.
- `myseq.physdrum~`: nonlinear membrane and eight-mode percussion model that morphs between skin, frame, plate, and fractured materials.
- `myseq.lsystem~`: polyphonic rewriting-grammar synthesizer whose branches control FM routing, ratios, drift, wavefolding, and generational depth.
- `myseq.growth~`: cellular oscillator colony with logistic growth, diffusion, mutation, cell division, coupling, and polyphonic emergence/collapse envelopes.

Suggested routing:

```text
megaseq outlet 4 -> myseq.physpluck~
megaseq outlet 1 -> myseq.physdrum~
megaseq outlet 4 -> myseq.lsystem~
megaseq outlet 4 -> myseq.growth~
```

Change `channel` or enable `omni 1` to connect them to another `megaseq` voice.

### Audio processors and modulators

- `myseq.multifilter~`: stereo state-variable bank with LP/BP/HP/notch morphing, dual formants, envelope/note following, nonlinear drive, and orbital cutoff motion.
- `myseq.multidelay~`: eight-tap stereo delay whose tap positions behave as spring-mass orbits inside a cross-feedback diffusion matrix.
- `myseq.bufferlab~`: eight-second multihead buffer with scanning, variable speed, reverse populations, granular offsets, smear, jitter, feedback, and freeze.
- `myseq.modfield~`: coupled-oscillator and Lorenz-attractor field; without input it generates stereo modulation/audio, while incoming audio can be ring/frequency modulated by the field.
- `myseq.resonator~`: twelve-mode stereo physical resonator that morphs between string, wood, plate, and cathedral-like structures; audio and MIDI events can excite it simultaneously.
- `myseq.biogate~`: reaction-diffusion multiband dynamics processor in which frequency-band gates grow, diffuse, mutate, compete, and decay.

Suggested audio chain:

```text
instrument~
-> myseq.multifilter~
-> myseq.multidelay~
-> myseq.bufferlab~
-> myseq.resonator~
-> myseq.biogate~
-> myseq.visualizer~
```

### Common messages

```text
60 110 650 3
bang
bypass 1
freeze 1
motion 1
mutate 1 0.45 0.22
xy 0.72 0.35 0.68
character random
randomize
param 1 0.75
set cutoff 0.62
channel 3
omni 0
clear
dump
```

Algorithm-specific parameter names can also be sent directly as normalized values, for example `feedback 0.72`, `growth 0.8`, `dispersion 0.55`, or `coupling 0.4`.

### Persistent presets

Each new module owns a separate 16-slot bank in `~/.myseq/presets/`. The bank stores all twelve manual parameters, XY position, morph mix, mutation depth/rate, and motion/mutation state.

```text
preset 1 store MyState
preset 1 recall
preset 1 clear
preset list
preset save
preset load
```

## `myseq.visualizer~`

An insertable stereo analysis instrument with transparent audio passthrough. Connect any mono or stereo `myseq` instrument to visualize it without changing the signal. A mono connection to inlet 1 is automatically copied to both outputs.

The interactive GUI provides six views:

- `OVERVIEW`: spectrum, waveform, phase, levels, and frequency bands in one dashboard.
- `SPECTRUM`: left/right logarithmic FFT with dominant frequency and spectral centroid.
- `SPECTROGRAM`: scrolling frequency-over-time color history.
- `SPRAY`: energy-driven particles positioned by frequency and amplitude.
- `PHASE`: stereo vectorscope, correlation, and mid/side behavior.
- `METERS`: peak, RMS, six frequency bands, stereo width, balance, and crest factor.

```text
myseq.visualizer~
mode overview
mode spectrum
mode spectrogram
mode spray
mode phase
mode meters
freeze 1
peakhold 1
reset
reference -72.
smoothing 0.72
trail 0.94
spray 0.72
hue 0.54
fft 2048
range 20 20000
dump
```

All modes, freeze, peak hold, FFT resolution, visual range, color, persistence, and particle density are editable directly from the GUI. Contextual tooltips explain every control.

## `myseq.timecode~`

A real-time synchronization generator with an interactive GUI. The left outlet produces sample-accurate LTC/SMPTE audio; the right outlet emits raw MIDI Time Code bytes and can be connected directly to `midiout`.

It supports 24, 25, 29.97 drop-frame, and 30 fps. MTC emits all eight Quarter Frame messages plus Full Time Code SysEx messages when starting, locating, changing frame rate, or receiving `fullframe`.

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

The LTC outlet is intended for a timecode input, a recording track, or synchronization hardware. Do not monitor it at normal listening levels. Max DSP must be enabled to generate LTC; the MTC clock can continue running while LTC is muted.

## `megaseq`

`megaseq` retains the complete `myseq` engine and reorganizes it as a radial composition instrument. It has five concentric rings and five independent outlets, from left to right:

1. kick, MIDI channel 1
2. snare, MIDI channel 2
3. hihat, MIDI channel 5
4. melody, MIDI channel 3
5. bass, MIDI channel 4

Each outlet emits lists compatible with the modular instruments:

```text
pitch velocity duration_ms channel
```

The sequencer includes pattern editing, Euclidean rhythms, spray, ratchets, scales, motifs, progressions, Markov transitions, Delaunay/Voronoi geometry, fluid and pollution systems, composer logic, arpeggiation, reversible automation, scenes, morphing, locks, and persistent banks.

The interaction model is radial:

- The five rings represent kick, snare, hihat, melody, and bass; steps can be painted directly.
- The central core edits root, scale, and all twelve pitch classes.
- The eight outer satellites are scenes; Alt-click stores and click recalls.
- The side navigator organizes voice, section, music page, automation, diagnostics, and bank controls.
- The lower deck presents the active control bank as one compact row without covering the radial editor.
- The inner energy ring edits the 16 points of the phrase curve.

### Hybrid Rhythm Lab

The `RHYTHM` page combines radial and linear sequencing. It keeps the five radial orbits while opening a compact `Rhythm Ribbon` below the circle for precise step editing. Colored cells display the algorithmic prediction; white markers show the manual pattern. Both layers are blended by the trigger engine.

Each voice can run an independent rhythm engine:

- `original`: preserves the classic `myseq` behavior.
- `euclid`: distributes pulses evenly across the cycle.
- `polymeter`: reshapes the distribution with an independent cycle length per voice.
- `clave`: applies syncopated rhythmic skeletons adapted to each voice.
- `cellular`: produces evolving patterns with cellular automata.
- `burst`: releases event clusters near phrase boundaries.
- `hybrid`: combines Euclidean, clave, and cellular votes with dynamic accents.

`PULSE COUNT`, `POLYMETER CYCLE`, `ORBIT ROTATION`, `RHYTHM VARIATION`, `SYNCOPATION`, `GHOST NOTES`, and `PATTERN EVOLUTION` are specific to the selected voice. Scenes, morphing, and `savebank` preserve the complete rhythm-engine state.

Basic modular routing:

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

Receives the event generated by the sequencer's kick outlet:

```text
pitch velocity duration_ms channel
```

Example:

```text
36 116 260 1
```

The instrument provides a resizable GUI, eight voices, transient FM synthesis, a tonal envelope, filtered-noise click, saturation, and 2x oversampling. `clear` or `panic` stops every voice; `bang` triggers the default kick.

Short sequencer gates do not cut off the kick tail. `decay` defines its minimum duration, while `duration_ms` can extend it.

Main parameters:

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

Parameters can also be supplied as object attributes:

```text
myseq.kick~ @decay 320 @fmindex 3.2 @drive 0.45
```

### Interface and mutation

The GUI includes:

- A real-time oscilloscope and output meter.
- An XY field between `DEEP`, `PUNCH`, `CLICK`, and `METAL` identities.
- A visible automatic-mutation trajectory.
- Ten editable body, pitch, FM, click, drive, and level controls.
- Separate markers for the manual value and the effective post-morph value.
- Mutation amount, rate, and morph-mix controls.
- Audition, mutation, freeze, and randomization buttons.

Drag controls horizontally. Drag inside the XY field to change character. Double-click a control to restore its default value.

Performance messages:

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

After rebuilding a version that Max has already loaded, restart Max so it releases the previous `.mxo` from memory and loads the new GUI.

The header displays `DSP ON` when the instrument belongs to an active audio chain. If it displays `DSP OFF`, enable `ezdac~` or check the signal connection. Events received while DSP is disabled are not accumulated in the queue.

## `myseq.snare~`

Receives the second `myseq` outlet (`pitch velocity duration_ms channel`, channel 2). It combines two resonant modes, filtered noise, transient snap, rattle, FM, saturation, eight voices, and 2x oversampling.

The GUI interpolates between four identities:

- `TIGHT`: short, precise, and strongly articulated.
- `WOOD`: low modal body with less noise.
- `AIR`: a wide noise tail with mobile rattle.
- `METAL`: inharmonic ratios and intense FM.

In addition to continuous mutation, `VARIATION` produces small deterministic changes in modal tuning, color, and rattle for every hit.

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

The object requires no arguments:

```text
myseq.snare~
```

Connect the second `myseq` outlet to `myseq.snare~`.

## `myseq.melody~`

A stereo melodic synthesizer for the fourth `myseq` outlet (MIDI channel 3). It is 12-voice polyphonic and combines an FM/wavefold core with up to 128 grains, tuned dispersion, per-grain panning, and stereo cross-delay. Internal nonlinear processing uses 2x oversampling.

The four XY identities are:

- `GLASS`: crystalline, consonant, bright grains.
- `BLOOM`: soft attacks, long tails, and a warm tonal center.
- `DUST`: short, scattered, pointillistic granular clouds.
- `ORBIT`: more radical FM, fold, drift, and feedback.

`HARMONICITY` interpolates between consonant and fractured intervals, allowing the instrument to move into avant-garde territory without completely losing its musical anchor. Each note captures the effective morph state and can coexist with older notes.

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

The object requires no arguments:

```text
myseq.melody~
```

Attributes are also accepted at creation time:

```text
myseq.melody~ @density 36 @grainsize 120 @harmonicity 0.9 @feedback 0.3
```

Connect `myseq` outlet 4 to its inlet and route both signal outlets to the left and right channels. Enable `ezdac~` before starting the sequencer.

## `myseq.mega~`

A 16-voice polyphonic stereo synthesizer for `myseq` MIDI-event lists. Each voice combines a six-operator FM network with four topologies and a mega-oscillator containing sine, triangle, sub, and unison mass. It also provides wavefolding, ring modulation, deterministic chaos, a resonant filter, transients, stereo dimension, and four short glitch processes. The nonlinear core uses 2x oversampling.

`INTELLIGENCE` pulls operator ratios and mutation toward stable harmonic proportions. At lower values, `RATIO SPREAD`, `VOICE MUTATION`, `CHAOS`, and `GLITCH` can create more fractured structures without changing the incoming MIDI pitch.

XY-field organisms:

- `GRID`: precise articulation, consonant ratios, and restrained rhythmic glitches.
- `SWARM`: mega-oscillator, unison, detune, and stereo dimension.
- `CRYSTAL`: bright FM, wavefolding, and musical resonance.
- `FRACTURE`: complex topologies, ring modulation, chaos, and aggressive glitch.

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

It receives `pitch velocity duration_ms channel` and listens to channel 3 by default. The GUI includes 20 controls, a stereo oscilloscope, voice/glitch counters, persistent presets, tooltips, and visual feedback for every button.

## `myseq.hihat~`

A mutable stereo hi-hat for the third `myseq` outlet (MIDI channel 5). It combines six inharmonic metallic oscillators, ring modulation, FM, noise with a dynamic high-pass filter, transient strike, sizzle, saturation, and 2x oversampling.

`OPENNESS` interpolates between the `CLOSED DECAY` and `OPEN DECAY` envelopes. A closed hit automatically chokes previous open tails; a long MIDI duration can also open the articulation.

XY-field identities:

- `TIGHT`: closed, bright, precise, and centered.
- `SILK`: soft air, wide noise, and stereo spread.
- `TRASH`: rough, dark, saturated metal.
- `ALIEN`: intense FM, unusual tuning, and spatial motion.

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

The object requires no arguments:

```text
myseq.hihat~
```

It can also be configured at creation time:

```text
myseq.hihat~ @openness 0.2 @brightness 0.85 @inharmonicity 0.9 @spread 0.7
```

Connect `myseq` outlet 3 to `myseq.hihat~` and route its two signal outlets left and right.

## `myseq.bass~`

A polyphonic stereo bass for the fifth `myseq` outlet (MIDI channel 4). It combines a mono-compatible sub, stereo triangle/saw body, FM synthesis, wavefolding, a resonant four-stage filter, filter envelope, saturation, and glide.

XY-field identities:

- `SUB`: clean, centered, deep bass.
- `FUNK`: short attack, dynamic filter, and percussive body.
- `REESE`: wide, detuned, folded, and aggressive.
- `ACID`: high resonance, filter modulation, and pronounced glide.

```text
character sub
character funk
character reese
character acid
morph 0.3 0.75 1.
mutate 1 0.45 0.09
preset 1 store MyBass
preset 1 recall
```

Connect `myseq` outlet 5 to `myseq.bass~`; its two signal outlets are left and right.

## Persistent user presets

The five instruments, `myseq.fx~`, and `myseq.sprayverb~` each provide 16 independent slots. Presets retain all manual parameters, the performance/morph field, and motion or mutation settings. Default banks are stored in `~/.myseq/presets/` and load automatically when an object is created.

```text
preset 1 store MySound
preset 1 recall
preset 1 clear
preset list
preset save
preset load
```

`preset store` and `preset clear` immediately write the default bank. Add a path to export or import another bank:

```text
preset save /path/to/my_basses.bank
preset load /path/to/my_basses.bank
```

The same protocol works in `myseq.kick~`, `myseq.snare~`, `myseq.hihat~`, `myseq.melody~`, `myseq.bass~`, `myseq.fx~`, and `myseq.sprayverb~`.

## Contextual tooltips

The instrument and effect interfaces display contextual help after the pointer remains over an element for approximately 260 ms. Tooltips include the effective value and explain synthesis or processing controls, the XY field, characters, performance buttons, all 16 preset slots, and the lower macro controls.

Tooltips are enabled by default. Disable them from the Inspector or at object creation:

```text
myseq.kick~ @tooltips 0
myseq.snare~ @tooltips 0
myseq.hihat~ @tooltips 0
myseq.melody~ @tooltips 0
myseq.bass~ @tooltips 0
myseq.fx~ @tooltips 0
myseq.sprayverb~ @tooltips 0
myseq.timecode~ @tooltips 0
```

## `myseq.fx~`

A stereo multi-effect processor for live performance. Insert it after any instrument. It maintains an eight-second circular buffer for stutter, freeze, and glitch captures, and also provides continuous reverse, speed jitter, feedback, bit crushing, decimation, ring modulation, filtering, stereo expansion, drive, and dry/wet control.

It has two signal inputs and two signal outputs. For a mono source, connect only the left input; the external automatically copies it to both channels.

Performance actions:

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

The XY field blends manual parameters with a fragmentation/destruction macro; `MOTION` automatically travels through that field. `STUTTER` and `FREEZE` remain latched until pressed again, while `GLITCH` is momentary.

`myseq.fx~` also provides 16 independent persistent presets, contextual tooltips, and the same `STORE`, `SAVE`, and `LOAD` controls as the instruments.

## `myseq.sprayverb~`

A stereo granular reverb with a particle visualizer driven by real grain-emission events. It combines up to 96 grains, stereo history, shimmer, modulation, and an eight-line FDN.

Its `BREATH` system creates generative atmospheric space by lowering cloud density and intensity, exposing the dry material, and then automatically releasing a granular spray. `SPACE` manually triggers a void-and-release cycle; `BURST` releases the cloud immediately.

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

It provides stereo input and output, automatic mono copying from the left input, 16 persistent presets, tooltips, and four base atmospheres. `FREEZE` stops new input and keeps the diffusion network close to an infinite tail.

Each GUI also includes two rows of eight preset slots. Click a slot to recall it; press `STORE` and then a slot to capture the current state. Alt-click stores directly. `SAVE` writes the persistent bank, and `LOAD` reads it from disk.

## Standalone DSP tests

The tests do not require the Max SDK:

```sh
cmake -S . -B build-test -DMYSEQ_BUILD_EXTERNALS=OFF
cmake --build build-test
ctest --test-dir build-test --output-on-failure
```

## Building the externals

CMake automatically detects common Max SDK installations, including `/Applications/max-sdk-main`. If it cannot find a local copy, it fetches Cycling '74's `max-sdk-base`. An explicit path can also be supplied:

```sh
cmake -S . -B build -DMAX_SDK_BASE_DIR=/path/to/max-sdk-base
cmake --build build --config Release
```

On macOS, universal Intel/Apple Silicon bundles are generated in `externals/`.
