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
<div>
  <input id={name} type="number" {min} {max} {step} bind:value />
  <small>{unit}</small>
</div>
<input type="range" {min} {max} {step} bind:value />

<style>
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
