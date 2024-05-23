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
<small>
  <input id={name} type="number" {min} {max} {step} bind:value />
  {unit}
</small>

<style>
  small {
    color: #fff9;
    font-variant-numeric: tabular-nums;
  }
  [type="number"] {
    background: none;
    border: solid 1px #fff1;
    padding: 0.5em 0.75em;
    &:focus {
      border-color: #fff4;
      outline: none;
    }
  }
</style>
