<script>
  import Widget from "./Widget.svelte";

  let props = [
    {
      name: "icon-stroke",
      label: "Icon Stroke",
      value: 1.5,
      min: 0.5,
      max: 3,
      step: 0.01,
      unit: "px",
    },
  ];
</script>

<Widget {props}>
  <div class="buttons">
    <button class="primary">
      <svg
        xmlns="http://www.w3.org/2000/svg"
        fill="none"
        viewBox="0 0 24 24"
        stroke-width="1.5"
        stroke="currentColor"
        class="w-6 h-6"
      >
        <path
          stroke-linecap="round"
          stroke-linejoin="round"
          d="M6 12 3.269 3.125A59.769 59.769 0 0 1 21.485 12 59.768 59.768 0 0 1 3.27 20.875L5.999 12Zm0 0h7.5"
        />
      </svg>
      Send
    </button>
    <button>
      <svg
        xmlns="http://www.w3.org/2000/svg"
        fill="none"
        viewBox="0 0 24 24"
        stroke-width="1.5"
        stroke="currentColor"
        class="w-6 h-6"
      >
        <path
          stroke-linecap="round"
          stroke-linejoin="round"
          d="M12 6v6h4.5m4.5 0a9 9 0 1 1-18 0 9 9 0 0 1 18 0Z"
        />
      </svg>
      Schedule
    </button>
    <button>
      <svg
        xmlns="http://www.w3.org/2000/svg"
        fill="none"
        viewBox="0 0 24 24"
        stroke-width="1.5"
        stroke="currentColor"
        class="w-6 h-6"
      >
        <path
          stroke-linecap="round"
          stroke-linejoin="round"
          d="M19.5 14.25v-2.625a3.375 3.375 0 0 0-3.375-3.375h-1.5A1.125 1.125 0 0 1 13.5 7.125v-1.5a3.375 3.375 0 0 0-3.375-3.375H8.25m2.25 0H5.625c-.621 0-1.125.504-1.125 1.125v17.25c0 .621.504 1.125 1.125 1.125h12.75c.621 0 1.125-.504 1.125-1.125V11.25a9 9 0 0 0-9-9Z"
        />
      </svg>
      Save Draft
    </button>
  </div>
</Widget>

<style>
  .buttons {
    display: flex;
    gap: 1em;
  }
  button {
    --h: 180;
    --c: 3%;
    --l-start-default: 42%;
    --l-end-default: 34%;

    --l-start: var(--l-start-default);
    --l-end: var(--l-end-default);

    --l-change-hover: 1.14;
    --l-change-active: 0.94;

    --start-color: oklch(var(--l-start) var(--c) var(--h));
    --end-color: oklch(var(--l-end) var(--c) var(--h));

    font-family: system-ui;
    font-size: 1em;
    font-weight: 500;
    padding: 0.5em 1em;
    border-radius: 0.25em;
    border: none;
    box-shadow: inset 0 1px 0 #fff3;
    background: linear-gradient(var(--start-color), var(--end-color));
    color: white;
    cursor: pointer;
    display: flex;
    align-items: center;
    gap: 0.375em;
  }
  button:has(svg) {
    padding-inline-start: 0.75em;
  }
  button svg {
    height: 1.25em;
    color: #fffb;
    stroke-width: var(--icon-stroke);
  }
  button.primary {
    --c: 100%;
  }
  button:hover {
    --l-start: calc(var(--l-start-default) * var(--l-change-hover));
    --l-end: calc(var(--l-end-default) * var(--l-change-hover));
  }
  button:active {
    --l-start: calc(var(--l-start-default) * var(--l-change-active));
    --l-end: calc(var(--l-end-default) * var(--l-change-active));
  }
</style>
