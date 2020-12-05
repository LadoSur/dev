Gibber Music Notes
===

###### 2020-11-27

I just discovered Gibber.
The documentation seems pretty spartan and maybe this is because of it being a new project.

Gibber looks like a very good candidate to hack in.
It's Javascript based, has a lot of nice sounding synths "out of the box" and focuses
on effects.

I have to play around with it but so far it looks like Gibber might be much more accessible
in terms of ease of use and musical polish.

These are notes on my discovery of how to use the system.
As with many of these libraries, often times "the code is the documentation" so
I shouldn't be shy about diving into the code to figure out how things work.

I'll be focusing on the '[alpha](https://gibber.cc/alpha/playground/)' release as this seems to have some features and maybe
is a bit more accessible then the older version.

---

| | |
|---|---|
| Run code | `Alt` + `Enter` |
| Stop code | `Ctrl` + `.` |

Note that the `Alt` + `Enter` will run a block of code only, where each block
is marked by an empty line.

For example:

```
bass = Synth('bleep', {"decay":1})
  .note.seq( [0,1,2,3,4,5,6,7,8,9,10,11,12,13,14,15], 1 )

hat = Hat({ decay:.0125 })
  .trigger.seq( [1,.5], 1)
```

Will only either play the synth part or the drum part depending on where the cursor is, whereas

```
bass = Synth('bleep', {"decay":1})
  .note.seq( [0,1,2,3,4,5,6,7,8,9,10,11,12,13,14,15], 1 )
 // a space here!
hat = Hat({ decay:.0125 })
  .trigger.seq( [1,.5], 1)
```

will play the both.

`Gibber.clear` is the programmatic way of stopping sound.

---

By default, Gibber runs note sequences in a musical mode (e.g. 'aeolian') which
means note numbers reference the position in the musical mode.

Gibber controls this via the `Theory` object.
To get control, set the musical mode to `chromatic` like so:

```
Theory.mode = "chromatic"
bass = Synth('bleep', {"decay":1})
  .note.seq( [0,1,2,3,4,5,6,7,8,9,10,11],1)
```

See [theory.js](https://github.com/charlieroberts/gibber.audio.lib/blob/8820baa90f5a789eebe9b56c43adfa6f853996dd/js/theory.js) for details.

Note, as a sanity check, you can use the web application [pitchdetect](https://webaudiodemos.appspot.com/pitchdetect/) ([src](https://github.com/cwilso/pitchdetect)) to detect frequencies of notes being produced.

---

Here is a quick snippet to fade out the gain:

```
bass = Monosynth('bass')
bass.note.seq( [0,7], 1/4 )
bass.gain.fade(1,0,8);
```

This works to stop `bass` from playing:

```
bass.stop();
```

There's a way to do this in the old version of Gibber (with future?) and I suspect with the new Gibber as well but
here's a hacky way to do this:

```
bass = Monosynth('bass')
bass.note.seq( [0,7], 1/4 )

setTimeout( function() { bass.gain.fade( bass.gain.value, 0, 2); }, 1000);
setTimeout( function() { bass.stop(); }, 8000);
```
---

There's a `future` function that looks to maybe do what I want.

The problem I was running into is that the audio thread is working in it's own sectioned off environment (web worker?)
so it doesn't have a lot of context.

From  `charlieroberts` in a `TOPLAP` discussion, this schedules an event for 1s in the future (presumably because of 44100 sample rate):

```
Clock.bpm = 120
Theory.root = 'd#4'
Theory.mode = 'dorian'
  
verb  = Bus2( 'spaceverb' )
delay = Bus2( 'delay.1/6' )
  
bass = Synth('acidBass2', { saturation:20, gain:.3 })
  .connect( delay, .25 )
  
bass.note.tidal( '0 0 0 0 4 6 0 ~ 0 ~ 7 -7 ~ 0 -7 0' )
bass.decay.seq( [1/32, 1/16], 1/2 )
bass.glide.seq( [1,1,100,100 ], 1/4 )
bass.Q = gen( 0.5 + cycle(0.1) * 0.49 )
bass.cutoff = gen( 0.5 + cycle(0.07) * 0.45 ) 
bass.gain = 0.0 

future( s => {s.gain = 0.2;}, 44100, { s:bass });
```

Doing the following doesn't work (and crashes the audio thread?):

```
...
future( s => { s.gain.fade( a.gain.value, 0.2, 1; }, 44100, { s:bass });
```

---

Gibber comes with presets of instruments but I can't find documentation for them anywhere.
See the [presets directory](https://github.com/charlieroberts/gibber.audio.lib/tree/main/js/presets),
in particular, the [synth presets](https://github.com/charlieroberts/gibber.audio.lib/blob/main/js/presets/synth_presets.js).

Here are some table for ease of perusal:

---

| Synth Presets | Synth Presets | Synth Presets |
|---|---|---|
| acidBass | acidBass2 | bleep.dry |
| bleep | bleep.echo | shimmer |
| stringPad | cry | brass |
| brass.short | pwm.squeak | pwm.short |
| chirp | square.perc | square.perc.long |
| rhodes | | |


([synth_presets.js](https://github.com/charlieroberts/gibber.audio.lib/blob/main/js/presets/synth_presets.js))

---

| FM Presets | FM Presets | FM Presets |
|---|---|---|
| bass  | kick | bass.electro |
| glockenspiel | glockenspiel.short | frog  |
| gong  | drum  | |

([fm_presets.js](https://github.com/charlieroberts/gibber.audio.lib/blob/main/js/presets/fm_presets.js))

---


| MonoSynth Presets | MonoSynth Presets | MonoSynth Presets |
|---|---|---|
|  short.dry | arpy | lead |
|  lead2 | dirty | winsome |
|  pluckEcho | bassPad | warble |
|  dark | bass | bass2 |
|  edgy | easy | easyfx |
|  chords | chords.short | jump |
|  shinybass2 | shinybass | bass.muted |
|  short | noise |  |

([monosynth_presets.js](https://github.com/charlieroberts/gibber.audio.lib/blob/main/js/presets/monosynth_presets.js))

---

old version presets:

| FM |
|---|
| bass |
| brass |
| clarinet |
| drum |
| drum2 |
| frog |
| glockenspiel |
| gong |
| noise |
| radio |
| stabs |

| Mono |
|---|
| bass |
| bass2 |
| dark |
| dark2 |
| easy |
| easyfx |
| edgy |
| lead |
| noise |
| short |
| winsome |

| Synth |
|---|
| bleep |
| bleepEcho |
| calvin |
| cascade |
| rhodes |
| short |
| warble |

| Synth2 |
|---|
| pad2 |
| pad4 |

Buses connect multiple inputs or outputs to single effect.
For example, it might be a desired effect to twiddle a single parameter that then has an effect on multiple
instruments.
To "bus" these signals together, a `Bus` (or `Bus2` for stereo?) can be used.

In other words, Buses allow for fan-in/fan-out operations.

---




References
---

* [Gibber alpha playground](https://gibber.cc/alpha/playground/)
* [Gibber documentation](https://gibber.cc/alpha/playground/docs/index.html#prototypes-ugen)
* [Tutorial Vid](https://www.youtube.com/watch?v=hqWIdaAjdmI)
* [github.com/charlieroberts/gibber.audio.lib](https://github.com/charlieroberts/gibber.audio.lib)
* [TidalCycles](https://tidalcycles.org/index.php/Userbase)
* [Gibber User Manual](https://bigbadotis.gitbooks.io/gibber-user-manual/content/index.html)
