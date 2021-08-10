<script>
  import { onDestroy } from "svelte";
  import Tone from "tone";

  let hovered = false;
  let clicked = false;
  let top = 0;
  let left = 0;
  let playing = false;

  const synth = new Tone.FMSynth().toMaster();
  const pitchInterpolater = new Tone.CtrlInterpolate([40, 2000]);
  const volumeInterpolater = new Tone.CtrlInterpolate([5, -20]);

  onDestroy(() => {
    synth.dispose();
    pitchInterpolater.dispose();
    volumeInterpolater.dispose();
  });

  function handleMouseOver(e) {
    hovered = true;
  }

  function handleMouseOut(e) {
    hovered = false;
    clicked = false;
    top = 0;
    left = 0;
  }

  function handleMouseDown(e) {
    clicked = true;
  }

  function handleMouseUp(e) {
    clicked = false;
  }

  function handleMouseMove({ target, clientX, clientY }) {
    const element = target;
    left = (clientX - element.offsetLeft) / element.offsetWidth;
    top = (clientY - element.offsetTop) / element.offsetHeight;
  }

  $: {
    pitchInterpolater.index = left;
    volumeInterpolater.index = top;
    const pitch = pitchInterpolater.value;
    const volume = volumeInterpolater.value;

    if (playing) {
      synth.volume.value = volume;
      synth.setNote(pitch);
    }

    if (clicked && !playing) {
      synth.triggerAttack(pitch);
      playing = true;
    }

    if (playing && !clicked) {
      synth.triggerRelease();
      playing = false;
    }
  }
</script>

<style>
  main {
    display: flex;
    align-items: center;
    justify-content: center;
    width: 100vw;
    height: 100vh;
    flex-direction: column;
    font-family: sans-serif;
  }

  .control-surface {
    width: 600px;
    height: 600px;
    border: 5px solid lightgray;
    border-radius: 10px;
    cursor: crosshair;
    background-color: darkgray;
  }

  .control-surface.active {
    border-color: green;
  }
</style>

<main>
	<h1>Svelte Theremin</h1>
	<div
    class="control-surface"
    class:active={clicked}
    on:mouseover={handleMouseOver}
    on:mouseout={handleMouseOut}
    on:mousedown={handleMouseDown}
    on:mouseup={handleMouseUp}
    on:mousemove={handleMouseMove}
  >
  </div>
</main>