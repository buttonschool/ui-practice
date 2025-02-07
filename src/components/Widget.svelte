<script>
  import Range from "./Range.svelte";
  export let props;

  $: style = props
    .map(({ name, value, unit }) => `--${name}: ${value}${unit};`)
    .join("");
</script>

<div class="exercise">
  <div class="canvas" {style}>
    <slot></slot>
  </div>

  <div class="controls">
    {#each props as { value, min, max, step = 1, unit = "", name, label = name }}
      <Range bind:value {min} {max} {step} {name} {unit} {label} />
    {/each}
  </div>
</div>

<style>
  .exercise {
    border: 1px solid #fff3;
    border-radius: 0.5em;
    overflow: hidden;
  }
  .canvas {
    display: flex;
    aspect-ratio: 16 / 9;
    justify-content: center;
    align-items: center;
    background-color: #222;
  }
  .controls {
    border-block-start: 1px solid #fff2;
    display: grid;
    grid-template-columns: minmax(6em, max-content) 1fr 6em;
    grid-auto-rows: min-content;
    gap: 2em 1em;
    padding: 2em;
    align-items: center;
  }
  @media screen and (min-width: 1024px) {
    .exercise {
      display: grid;
      grid-template-columns: 2fr 1fr;
    }
    .controls {
      border-block-start: none;
      border-inline-start: 1px solid #fff2;
    }
  }
</style>
