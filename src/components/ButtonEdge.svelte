<script>
  import Widget from "./Widget.svelte";

  let props = [
    {
      name: "l-change-edge",
      label: "Edge Shade",
      value: 0.5,
      min: 0,
      max: 1,
      step: 0.01,
      unit: "",
    },
    {
      name: "edge-size-default",
      label: "Height",
      value: 6,
      min: 0,
      max: 16,
      step: 0.1,
      unit: "px",
    },
    {
      name: "well-offset",
      label: "Well Offset",
      value: 2,
      min: 1,
      max: 10,
      step: 0.1,
      unit: "px",
    },
    {
      name: "rim-highlight",
      label: "Rim highlight",
      value: 0.025,
      min: 0,
      max: 1,
      step: 0.005,
      unit: "",
    },
  ];
</script>

<Widget {props}>
  <div class="group">
    <button> Subscribe </button>
  </div>
</Widget>

<style>
  .group {
    display: flex;
    gap: 1em;
  }
  button {
    --h: 25deg;
    --c: 100%;
    --l: var(--l-default);

    --l-default: 40%;
    --l-change-hover: 1.15;
    --l-change-active: 1.05;
    /* --l-change-edge: 0.8; */

    --l-edge: calc(var(--l) * var(--l-change-edge));

    --color-background: oklch(var(--l) var(--c) var(--h));
    --color-edge: oklch(var(--l-edge) var(--c) var(--h));

    /* --edge-size-default: 6px; */
    --press-amount: 0px;
    /* --well-offset: 2px; */
    /* --rim-highlight: 0.025; */

    --edge-size: calc(var(--edge-size-default) - var(--press-amount));

    font-family: system-ui;
    font-size: 1.2em;
    font-weight: 700;
    padding: 0.625em 1.125em;
    border-radius: 0.25em;
    border: none;
    box-shadow:
      inset 0 1px 0 #fff2,
      0 var(--edge-size) 0 var(--color-edge),
      0 calc(var(--edge-size) - 1px) 0 calc(var(--well-offset) + 1px) #111,
      0 calc(var(--edge-size) - 1px) 0 calc(var(--well-offset) + 2px)
        rgba(255, 255, 255, var(--rim-highlight));
    background: var(--color-background);
    color: #fffd;
    text-shadow: 0 -1px #0003;
    translate: 0 var(--press-amount);
    cursor: pointer;
    display: flex;
    align-items: center;
    gap: 0.375em;
  }
  button:hover {
    --l: calc(var(--l-default) * var(--l-change-hover));
  }
  button:active {
    --l: calc(var(--l-default) * var(--l-change-active));
    --press-amount: 3px;
  }
</style>
