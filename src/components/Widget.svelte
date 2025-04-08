<script>
  import Range from "./Range.svelte";
  export let props;

  $: style = props
    .map(({ name, value, unit }) => `--${name}: ${value}${unit};`)
    .join("");

  $: grouped_props = Object.values(
    props.reduce((acc, prop) => {
      if (!acc[prop.group]) acc[prop.group] = { group: prop.group, props: [] };
      acc[prop.group].props.push(prop);
      return acc;
    }, {})
  );
</script>

<div class="exercise">
  <div class="canvas" {style}>
    <slot></slot>
  </div>

  <div class="controls">
    {#each grouped_props as { group, props }}
      {#if group}
        <details open class="group">
          <summary>{group}</summary>
          <div class="controls-layout">
            {#each props as { value, min, max, step = 1, unit = "", name, label = name }}
              <Range bind:value {min} {max} {step} {name} {unit} {label} />
            {/each}
          </div>
        </details>
      {:else}
        {#each props as { value, min, max, step = 1, unit = "", name, label = name }}
          <Range bind:value {min} {max} {step} {name} {unit} {label} />
        {/each}
      {/if}
    {/each}
  </div>
</div>

<style>
  .exercise {
    background-color: #222;
    border: 1px solid #fff1;
    border-radius: 0.5em;
    overflow: hidden;
    box-shadow:
      0 0.5em 1em -0.5em #0008,
      0 0.5em 2em #0004;
  }
  .canvas {
    display: flex;
    aspect-ratio: 16 / 9;
    justify-content: center;
    align-items: center;
    background-color: #191919;
  }
  .controls,
  .controls summary {
    border-block-start: 1px solid #fff1;
  }
  .controls summary {
    margin-top: -1px;
  }
  .controls,
  .controls-layout {
    display: grid;
    grid-template-columns: max-content 1fr max-content;
    grid-auto-rows: min-content;
    gap: 2em 1.5em;
    padding: 2em;
    align-items: center;
  }
  .controls-layout {
    padding-block: 0.5em 1.5em;
    gap: 1em 1.5em;
  }
  .controls details {
    grid-column: 1 / -1;
    margin: -2em -2em 0;
  }
  .controls summary {
    color: #888;
    cursor: pointer;
    font-size: smaller;
    font-weight: bold;
    letter-spacing: 0.05em;
    padding: 1.5em 1rem 1.5em;
    text-transform: uppercase;
    -webkit-user-select: none; /* Safari */
    -ms-user-select: none; /* IE 10+ */
    user-select: none; /* Standard */
  }
  @media screen and (min-width: 1024px) {
    .exercise {
      display: grid;
      grid-template-columns: 2fr 1fr;
    }
    .controls {
      border-block-start: none;
      border-inline-start: 1px solid #fff1;
    }
  }
</style>
