<script>
  export let onKeyPressed;
  // export let onKeyReleased;
  if (navigator.requestMIDIAccess) {
    navigator.requestMIDIAccess().then(success);
  }

  function success(midiAccess) {
    console.log("YO MIDI");

    console.log(midiAccess);
    midiAccess.addEventListener("statechange", updateDevices);
    const inputs = midiAccess.inputs;
    inputs.forEach((input) => {
      console.log(input);
      input.onmidimessage = onKeyPressed;
    });
  }

  function updateDevices(event) {
    console.log(event);
  }
</script>
