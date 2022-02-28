<script>
  import { NOTES } from "./constants";
  import Keyboard from "./Keyboard.svelte";
  import Midi from "./Midi.svelte";
  let isReady = false;
  const oscillators = Array(3);
  let audioContext;
  const waveforms = ["sine", "square", "sawtooth", "triangle"];
  let currentNote = 440;
  let currentOscillatorType = 2;
  let gainNode;
  let detune = 0;
  let filterCutoff = 5000;
  let filter;
  const ADSR = { attack: 0, decay: 1, sustain: 1, release: 0.05 };
  const MAX_ADSR_STAGE_DURATION = 5;
  let currentNotes = [];
  const MAX_POLYPHONY = 1;
  const createOscillator = (frequency, detune) => {
    if (gainNode) gainNode.gain.cancelScheduledValues(audioContext.currentTime);

    const osc = audioContext?.createOscillator();
    osc.type = waveforms[currentOscillatorType];
    osc.frequency.value = frequency;
    osc.detune.value = detune;
    // osc.connect(audioContext.destination);
    // osc.start();
    return osc;
  };
  const noteOn = (note) => {
    // if (currentNotes.length > MAX_POLYPHONY) {
    //   const lastNoteOscillators = currentNotes.pop();
    //   lastNoteOscillators.forEach(osc=> osc.stop();
    // } else if (MAX_POLYPHONY === 1) {
    // noteOff();
    // }
    stop();
    const frequency = note;
    const unisonWidth = detune;
    oscillators[0] = createOscillator(frequency, 0);
    oscillators[1] = createOscillator(frequency, unisonWidth);
    oscillators[2] = createOscillator(frequency, -unisonWidth);
    oscillators[0].connect(gainNode);
    oscillators[1].connect(gainNode);
    oscillators[2].connect(gainNode);

    const now = audioContext.currentTime;
    const attackDuration = ADSR.attack * MAX_ADSR_STAGE_DURATION;
    const attackEndTime = now + attackDuration;
    const decayDuration = ADSR.decay * MAX_ADSR_STAGE_DURATION;

    gainNode.gain.setValueAtTime(0, audioContext.currentTime);
    gainNode.gain.linearRampToValueAtTime(1, attackEndTime);
    gainNode.gain.setTargetAtTime(ADSR.sustain, attackEndTime, decayDuration);
    // gainNode.gain.setValueCurveAtTime(
    //   ADSR.sustain,
    //   attackEndTime,
    //   decayDuration
    // );

    // oscillators.forEach((oscillator) => {
    //   oscillator.connect(filter);
    // });

    oscillators.forEach((osc) => osc.start());
    // currentNotes.push(oscillators);
  };
  const killOscillators = (t = 0) => {
    gainNode.gain.cancelScheduledValues(t);
    // console.log(filter);
    filter.frequency.cancelScheduledValues(t);
    oscillators.forEach((oscillator) => oscillator?.stop(t));
  };

  const noteOff = () => {
    const currentTime = audioContext.currentTime;
    const relDuration = ADSR.release * MAX_ADSR_STAGE_DURATION;
    killOscillators(currentTime + relDuration);

    const adsrTarget = gainNode.gain;
    adsrTarget.setValueAtTime(adsrTarget.value, currentTime);
    adsrTarget.linearRampToValueAtTime(0, currentTime + relDuration);
  };
  const init = () => {
    // osc1?.stop();
    audioContext = new (AudioContext || webkitAudioContext)();
    gainNode = audioContext.createGain();
    // gainNode.connect(audioContext.destination);
    filter = audioContext.createBiquadFilter();
    gainNode.connect(filter);
    // filter.connect(audioContext.destination);
    filter.type = "lowpass";
    filter.frequency.setValueAtTime(filterCutoff, audioContext.currentTime);
    const volumeNode = audioContext.createGain();
    volumeNode.gain.value = 0.4;
    // gainNode.connect(volumeNode);
    filter.connect(volumeNode);
    volumeNode.connect(audioContext.destination);
    // osc1 = audioContext.createOscillator();
    // osc1.type = waveforms[currentOscillatorType];
    // osc1.frequency.value = note || currentNote;
    // osc1.connect(audioContext.destination);
    // osc1.start();
    // osc1.stop(audioContext.currentTime + 2);
    isReady = true;
  };
  // const setOscillatorType = (type) => {
  //   if (currentOscillatorType) currentOscillatorType = type;
  // };
  const setCurrentNote = (note) => {
    // if (currentNote === note) {
    //   stop();
    //   currentNote = 0;
    //   return;
    // }
    currentNote = note;
    noteOn(note);
    // osc1.frequency.value = note;
    // osc1.start()
    // osc1.stop(audioContext?.currentTime + 2);
  };
  const setOscillatorType = () => {
    // if (osc1) osc1.type = waveforms[currentOscillatorType];
    oscillators.forEach((oscillator) => {
      if (oscillator) {
        oscillator.type = waveforms[currentOscillatorType];
      }
    });
  };
  const stop = () => {
    // console.log(oscillators);
    oscillators.forEach((oscillator) => oscillator?.stop());
  };

  function updateFilterValue() {
    filter.frequency.setValueAtTime(filterCutoff, audioContext.currentTime);
  }
  function setPatch(name) {
    switch (name) {
      case "pad":
        ADSR.attack = 0.5;
        ADSR.decay = 0.2;
        ADSR.sustain = 1;
        ADSR.release = 0.6;
        currentOscillatorType = 3;

        break;
      case "pluck":
        ADSR.attack = 0.01;
        ADSR.decay = 0.6;
        ADSR.sustain = 0;
        ADSR.release = 0.05;
        currentOscillatorType = 2;

        break;

      case "lead":
        ADSR.attack = 0;
        ADSR.decay = 1;
        ADSR.sustain = 1;
        ADSR.release = 0.05;
        currentOscillatorType = 1;

        break;
      default:
        break;
    }
  }
</script>

<main>
  <h1>Hello Svelte Synth!</h1>
  <!-- <p>
    Visit the <a href="https://svelte.dev/tutorial">Svelte tutorial</a> to learn
    how to build Svelte apps.
  </p> -->
  <button on:click={init}>INIT</button>
  {#if isReady}
    <button on:click={() => location.reload()}>Killswitch</button>
    <div>
      <button on:click={() => setPatch("pad")}>Pad</button>
      <button on:click={() => setPatch("pluck")}>Pluck</button>
      <button on:click={() => setPatch("lead")}>Lead</button>
    </div>
    <div>
      <p>Oscillator type: {waveforms[currentOscillatorType]}</p>
      <input
        type="range"
        bind:value={currentOscillatorType}
        min="0"
        max="3"
        on:input={setOscillatorType}
      />
    </div>
    <div>
      <input
        type="range"
        bind:value={filterCutoff}
        min="0"
        max="5000"
        on:input={updateFilterValue}
      />
      <p>Filter cutoff: {filterCutoff}</p>
    </div>
    <div>
      <p>Detune: {detune}</p>
      <input type="range" bind:value={detune} min="0" max="100" />
    </div>
    <div>
      <h2>ADSR</h2>
      <div class="adsr-section">
        <div>
          <h3>A</h3>
          <input
            type="range"
            bind:value={ADSR.attack}
            min="0"
            max="1"
            step="0.01"
          />
          <p>{ADSR.attack}</p>
        </div>
        <div>
          <h3>D</h3>
          <input
            type="range"
            bind:value={ADSR.decay}
            min="0"
            max="1"
            step="0.01"
          />
          <p>{ADSR.decay}</p>
        </div>
        <div>
          <h3>S</h3>
          <input
            type="range"
            bind:value={ADSR.sustain}
            min="0"
            max="1"
            step="0.01"
          />
          <p>{ADSR.sustain}</p>
        </div>
        <div>
          <h3>R</h3>
          <input
            type="range"
            bind:value={ADSR.release}
            min="0"
            max="1"
            step="0.01"
          />
          <p>{ADSR.release}</p>
        </div>
      </div>
    </div>

    <Keyboard
      onMouseDown={(note) => setCurrentNote(NOTES[note])}
      onMouseUp={noteOff}
    />
    <Midi
      onKeyPressed={(e) => {
        if (e.data[0] === 144) {
          // console.log(NOTES[Object.keys(NOTES)[e.data[1]]]);

          // NOTES[Object.keys(NOTES)[e.data[1]]];
          setCurrentNote(NOTES[Object.keys(NOTES)[e.data[1]]]);
        } else {
          noteOff(NOTES[Object.keys(NOTES)[e.data[1]]]);
        }
      }}
    />
  {/if}
</main>

<style>
  main {
    text-align: center;
    padding: 1em;
    max-width: 240px;
    margin: 0 auto;
  }

  h1 {
    color: #ff3e00;
    text-transform: uppercase;
    font-size: 4em;
    font-weight: 100;
  }

  @media (min-width: 640px) {
    main {
      max-width: none;
    }
  }
  .adsr-section {
    display: flex;
    justify-content: center;
  }
  .adsr-section input[type="range"] {
    -webkit-appearance: slider-vertical;
  }
</style>
