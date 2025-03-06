<script>
  export let label;
  export let name;
  export let min;
  export let max;
  export let step;
  export let value;
  export let unit;
</script>

<label for={name}>{label}</label>
<input type="range" {min} {max} {step} bind:value />
<div class="precise-input">
  <input id={name} type="number" {min} {max} {step} bind:value />
  {#if unit}<small>{unit}</small>{/if}
</div>

<style>
  .precise-input {
    display: grid;
    grid-template-columns: 1fr max-content;
    gap: 1em;
    align-items: center;
  }
  small {
    color: #fff9;
    font-variant-numeric: tabular-nums;
  }
  [type="number"] {
    background: #fff1;
    border: solid 1px #fff0;
    border-radius: 0.25em;
    padding: 0.5em 0.75em;
    &:hover {
      border-color: #fff2;
    }
    &:focus {
      border-color: aquamarine;
      outline: none;
    }
  }
</style>
