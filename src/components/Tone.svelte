<script>
  import {
    active,
    toneLoaded,
    enableMIDI,
    volumeVal,
    glide,
    toneOutput,
    scaleType,
    scaleNotes,
    tonic,
    oscillatorType,
    analyser,
    audioControls,
    startOctave,
    endOctave,
    currentMIDITitle,
    currentMIDI,
    midiList,
  } from "../stores.js";
  import { jsUcfirst, findNext } from "../helpers.js";
  import { tonicOrder, scales } from "../config.js";
  import Recorder from './Recorder.svelte';
//   import unmuteAudio from 'unmute-ios-audio';

//   let unmute = false;

  const generateScale = (tonic, key, octaves) => {
    let scale = teoria.scale(tonic, key);
    let newNotes = [];
    for (let i = 0; i < octaves; i++) {
      scale.notes().forEach((e) => {
        let scientificNot = e.scientific();
        let base = scientificNot.slice(0, -1);
        let num = parseInt(scientificNot.match(/(\d+)/)[0]) + i;
        newNotes.push(base + num);
      });
    }
    return newNotes;
  };

  $: {
    let tonicNote = $tonic.includes("/")
      ? $tonic.substring(0, $tonic.lastIndexOf("/"))
      : $tonic;
    let arr = generateScale(
      tonicNote + $startOctave,
      $scaleType.toLowerCase().replace(/\s/g, ""),
      $endOctave - $startOctave
    );
    arr.push(tonicNote + $endOctave);
    scaleNotes.set(arr);
  }

  const masterVolume = new Tone.Volume($volumeVal);
  const masterCompressor = new Tone.Compressor({
    ratio: 12,
    threshold: -28,
    release: 0.25,
    attack: 0.003,
    knee: 30,
  });
  const masterAnalyser = new Tone.Analyser("waveform", 64);
  analyser.set(masterAnalyser);

  const gain1 = new Tone.Gain(0.1);
  const gain2 = new Tone.Gain(0.1);

  const mainOsc = new Tone.OmniOscillator({
    frequency: 440,
    detune: 0,
    phase: 0,
    spread: 20,
    count: 3,
    modulationIndex: 2,
    modulationType: "square",
    harmonicity: 1,
    partialCount: 0,
  });
  $: masterVolume.volume.value = $volumeVal;
  $: mainOsc.type =
    $oscillatorType !== "Oscillator Off"
      ? $oscillatorType.toLowerCase().replace(/\s/g, "")
      : "sine";

  let vibrato = new Tone.Vibrato({
    maxDelay: 0.005,
    frequency: 5,
    depth: 0.05,
    type: "sine",
  });

  mainOsc.chain(gain1, Tone.Destination);
  gain2.connect(Tone.Destination);
  Tone.Destination.chain(masterCompressor, masterVolume, masterAnalyser);

  let midiSynths = [];
  let lastMIDI;
  let savedMIDI = null

  const initMidi = (url) => {
    Midi.fromUrl(url).then((midi) => {
      savedMIDI = midi;
      const now = 1;
      const nowDelay = 2;

      midi.tracks.forEach((track, i) => {
        const synth = new Tone.PolySynth({
          maxPolyphony : 60,
          options: {
            oscillator: {
              type:
                $oscillatorType === "Fat Sine"
                  ? "sine"
                  : $oscillatorType === "Fat Triangle"
                  ? "triangle"
                  : $oscillatorType === "Fat Sawtooth"
                  ? "sawtooth"
                  : $oscillatorType === "Fat Square"
                  ? "square"
                  : $oscillatorType === "PWM"
                  ? "pwm"
                  : $oscillatorType === "Pulse"
                  ? "pulse"
                  : $oscillatorType === "Oscillator Off"
                  ? "triangle"
                  : $oscillatorType.toLowerCase().replace(/\s/g, ""),
              spread: $oscillatorType.includes("Fat") ? 20 : 0,
              count: $oscillatorType.includes("Fat") ? 3 : 0,
              modulationFrequency: $oscillatorType === "PWM" ? 0.4 : 0,
              width: $oscillatorType === "Pulse" ? 0.2 : 0,
            },
            envelope: {
              attackCurve: "exponential",
              attack: 0.02,
              decayCurve: "exponential",
              decay: 0.1,
              sustain: 0.3,
              release: 1,
            },
          },
        }).connect(gain2);
        
        midiSynths.push(synth);
        //scheduleOnce all of the events
        track.notes.forEach((note) => {
          let duration = note.duration <= 0 ? 0.1 : note.duration;
          Tone.Transport.scheduleOnce((time) => {
            synth.triggerAttack(note.name, time, note.velocity);
            synth.triggerRelease(note.name, time+note.duration);
            // synth.triggerAttackRelease(
            //   note.name,
            //   duration,
            //   time,
            //   note.velocity
            // );
          }, note.time + now);
        });
      });

      Tone.Transport.scheduleOnce(() => {
        let midiArr = Object.keys($midiList)
        if(!$midiList.custom.url){
           midiArr.shift()
        }
        let nextItem = findNext(($currentMIDI.includes('custom') ? 'custom' : $currentMIDI), midiArr);
        currentMIDI.set(nextItem);
      }, nowDelay + midi.duration);

      midi.header.keySignatures.forEach((signature) => {
        Tone.Transport.scheduleOnce(() => {
          let scale =
            scales.find((a) => a.includes(jsUcfirst(signature.scale))) || null;
          if (scale) {
            scaleType.set(scale);
          }
        }, Tone.Time(now).toTicks() + signature.ticks + "i");

        Tone.Transport.scheduleOnce(() => {
          let key =
            signature.key.length > 1
              ? tonicOrder.find((a) => a.includes(signature.key))
              : signature.key;
          if (key) {
            tonic.set(key);
          }
        }, Tone.Time(now).toTicks() + signature.ticks + "i");
      });

      console.log("Current Playing: " + lastMIDI);
      Tone.Transport.start();
      currentMIDITitle.set((lastMIDI.includes('custom')?'custom':lastMIDI));
    });
  };

  const cleanupSynths = () => {
    // midiSynths[0].dispose()
    // midiSynths = []
    while (midiSynths.length) {
      const synth = midiSynths.shift();
      synth.dispose()
      // synth.disconnect();
    }
    Tone.Transport.stop();
    Tone.Transport.cancel(0);
  };

  $: {
    if ($enableMIDI && $currentMIDI) {
      if (lastMIDI !== $currentMIDI) {
        lastMIDI = $currentMIDI;
        if (midiSynths.length > 0) {
          cleanupSynths();
        }
        let url = lastMIDI.includes('custom') ? $midiList.custom.url : $midiList[lastMIDI].url
        initMidi(url);
      }
    } else {
      lastMIDI = null;
      if (midiSynths.length > 0) {
        cleanupSynths();
      }
    }
  }
  
  oscillatorType.subscribe(val=>{
    if($active && midiSynths.length>0 && $enableMIDI){
      
    let baseType =  val.includes("Sine")
            ? "sine"
            : val.includes("Triangle")
            ? "triangle"
            : val.includes("Sawtooth")
            ? "sawtooth"
            : val.includes("Square")
            ? "square"
            : val === "PWM"
            ? "sawtooth"
            : val === "Pulse"
            ? "square"
            : val === "Oscillator Off"
            ? "triangle"
            : "triangle"
      let prefix = val.includes("FM") ? '' 
      : val.includes("AM") ? 'am'
      : ''
      let partial = ''
          
      midiSynths.forEach((synth,j) => {
          
          synth.set({
            oscillator:{
              type:baseType,
              // type:
              //   val === "Fat Sine"
              //     ? "sine"
              //     : val === "Fat Triangle"
              //     ? "triangle"
              //     : val === "Fat Sawtooth"
              //     ? "sawtooth"
              //     : val === "Fat Square"
              //     ? "square"
              //     : val === "PWM"
              //     ? "pwm"
              //     : val === "Pulse"
              //     ? "pulse"
              //     : val === "Oscillator Off"
              //     ? "triangle"
              //     : val.toLowerCase().replace(/\s/g, ""),
              spread: val.includes("Fat") ? 20 : 0,
              count: val.includes("Fat") ? 3 : 0,
              modulationFrequency: val === "PWM" ? 0.4 : 0,
              width: val === "Pulse" ? 0.2 : 0
            }
          })
          
        })
    }
  })
  
  $: {
    if ($active) {
      
      // if(!unmute){
      //   unmute = true
      //   unmuteAudio()
      //   Tone.start();
      // }
      if (Tone.context.state !== "running") {
        Tone.start();
      }
      
      let maxFreq = Tone.Frequency($scaleNotes[$scaleNotes.length - 1]);
      let minFreq = Tone.Frequency($scaleNotes[0]);

      if (Tone.context.state === "running" && mainOsc.state === "stopped") {
        mainOsc.start();
      }

      let gain1Max =
        $oscillatorType === "Oscillator Off"
          ? 0
          : $oscillatorType === "Sine"
          ? 0.5
          : $oscillatorType === "FM Sine"
          ? 0.2
          : $oscillatorType === "AM Sine"
          ? 0.6
          : $oscillatorType === "Fat Sine"
          ? 0.6
          : $oscillatorType === "Square"
          ? 0.15
          : $oscillatorType === "FM Square"
          ? 0.15
          : $oscillatorType === "AM Square"
          ? 0.2
          : $oscillatorType === "Fat Square"
          ? 0.1
          : $oscillatorType === "Triangle"
          ? 0.6
          : $oscillatorType === "FM Triangle"
          ? 0.3
          : $oscillatorType === "AM Triangle"
          ? 0.5
          : $oscillatorType === "Fat Triangle"
          ? 0.5
          : $oscillatorType === "Sawtooth"
          ? 0.17
          : $oscillatorType === "FM Sawtooth"
          ? 0.2
          : $oscillatorType === "AM Sawtooth"
          ? 0.3
          : $oscillatorType === "Fat Sawtooth"
          ? 0.17
          : $oscillatorType === "PWM"
          ? 0.1
          : $oscillatorType === "Pulse"
          ? 0.1
          : 0.1;
      // let gain1Max = .1
      
      gain1.gain.value = $enableMIDI ? 0 : $audioControls.y * gain1Max;
      gain2.gain.value = gain1Max;
      vibrato.frequency.value = $audioControls.y * 10;

      let steps = {
        x: Math.min(
          ~~($audioControls.x / (1 / $scaleNotes.length)),
          $scaleNotes.length - 1
        ),
        y: Math.min(
          ~~($audioControls.y / (1 / $scaleNotes.length)),
          $scaleNotes.length - 1
        ),
      };
      let glideFreq = $audioControls.x * (maxFreq - minFreq) + minFreq;
      let calcFreq = $glide
        ? Math.floor(glideFreq)
        : Math.floor(Tone.Frequency($scaleNotes[steps.x]).toFrequency());

      mainOsc.frequency.value =
        mainOsc.frequency.value !== calcFreq
          ? calcFreq
          : mainOsc.frequency.value;

      let envelopeControl = {
        x: $enableMIDI ? steps.x / $scaleNotes.length : $audioControls.x,
        y: $enableMIDI ? steps.y / $scaleNotes.length : $audioControls.y,
      };
      let envelope = {
        attack: 0.005 + 0.2 * (1 - envelopeControl.x),
        sustain: 0.2 - 0.2 * envelopeControl.y,
        attackCurve: "exponential",
        decayCurve: "exponential",
        decay: 0.1 + 1.2 * (1 - envelopeControl.y),
        release: 0.1 + 1.2 * (1 - envelopeControl.x),
      }
      
      if (midiSynths.length > 0 && $enableMIDI) {
        const remap = (val,offset=.5) => {
          return Math.abs(val-offset)/offset
        }
        midiSynths.forEach((synth,j) => {
          synth.set({envelope: envelope});
        });
      }

      toneOutput.set({
        glide: $glide,
        total: $scaleNotes.length,
        index: steps,
        note: $scaleNotes[steps.x],
        freq: glideFreq,
        attack: envelope.attack,
        decay: envelope.decay
      });
    } else {
      if (mainOsc.state === "started") {
        mainOsc.stop();
      }
      if (Tone.context.state === "running") {
        Tone.getContext().rawContext.suspend();
      }
    }
  }

  toneLoaded.set(true);
</script>

{#if Tone && window.MediaRecorder}
<Recorder/>
{/if}