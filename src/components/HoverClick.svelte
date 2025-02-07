<script>
  import Widget from "./Widget.svelte";

  let props = [
    {
      name: "l-change-hover",
      label: "Hover",
      value: 1.5,
      min: 0.5,
      max: 1.5,
      step: 0.01,
      unit: "",
    },
    {
      name: "l-change-active",
      label: "Press",
      value: 0.5,
      min: 0.5,
      max: 1.5,
      step: 0.01,
      unit: "",
    },
  ];
</script>

<Widget {props}>
  <div class="stack">
    <div class="row">
      <button class="primary">Send</button>
      <button>Cancel</button>
    </div>
    <div class="row hovered">
      <button class="primary">Send</button>
      <button>Cancel</button>
    </div>
    <div class="row pressed">
      <button class="primary">Send</button>
      <button>Cancel</button>
    </div>
  </div>
</Widget>

<style>
  .stack {
    display: flex;
    flex-direction: column;
    gap: 2em;
  }
  .row {
    display: flex;
    gap: 1em;
  }
  button {
    --h: 200;
    --c: 3%;
    --l-start-default: 46%;
    --l-end-default: 32%;

    --l-start: var(--l-start-default);
    --l-end: var(--l-end-default);

    --start-color: oklch(var(--l-start) var(--c) var(--h));
    --end-color: oklch(var(--l-end) var(--c) var(--h));

    font-family: system-ui;
    font-size: 1.2em;
    font-weight: 500;
    padding: 0.4em 1em;
    border-radius: 0.25em;
    border: none;
    box-shadow: inset 0 1px 0 #fff5;
    background: linear-gradient(var(--start-color), var(--end-color));
    color: white;
    cursor: pointer;
  }
  button.primary {
    --c: 100%;
    --l-start-default: 53%;
    --l-end-default: 30%;
  }
  button:hover,
  .hovered button {
    --l-start: calc(var(--l-start-default) * var(--l-change-hover));
    --l-end: calc(var(--l-end-default) * var(--l-change-hover));
  }
  button:active,
  .pressed button {
    --l-start: calc(var(--l-start-default) * var(--l-change-active));
    --l-end: calc(var(--l-end-default) * var(--l-change-active));
  }
</style>
