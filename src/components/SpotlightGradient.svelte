<script>
  import Widget from "./Widget.svelte";

  let props = [
    {
      group: "Color Hues",
      name: "page-hue",
      label: "Canvas",
      value: 275,
      min: 0,
      max: 360,
      step: 5,
      unit: "",
    },
    {
      group: "Color Hues",
      name: "left-hue",
      label: "Left Spotlight",
      value: 25,
      min: 0,
      max: 360,
      step: 5,
      unit: "",
    },
    {
      group: "Color Hues",
      name: "right-hue",
      label: "Right Spotlight",
      value: 70,
      min: 0,
      max: 360,
      step: 5,
      unit: "",
    },
  ];
</script>

<Widget {props}>
  <div class="page">
    <div class="sheet"></div>
  </div>
</Widget>

<style>
  .page {
    --tilt: 15deg;
    --offset: 37.5%;
    /* --page-hue: 275; */

    background-color: oklch(27.5% 0.2 var(--page-hue));
    padding: 2rem;
    margin: 0;
    width: 100%;
    height: 100%;
  }
  .sheet {
    /* --left-hue: 25;
    --right-hue: 70; */

    --sheet-left: oklch(67.5% 0.25 var(--left-hue));
    --sheet-right: oklch(82.5% 0.18 var(--right-hue));

    aspect-ratio: 16 / 9;
    border-radius: 12px;
    background: radial-gradient(
        farthest-side at bottom left,
        var(--sheet-left),
        var(--sheet-left) 5%,
        transparent 85%
      ),
      radial-gradient(
        farthest-side at bottom right,
        var(--sheet-right),
        var(--sheet-right) 5%,
        transparent 85%
      ),
      linear-gradient(
        to bottom,
        oklch(22.5% 0.2 var(--page-hue)),
        oklch(37.5% 0.2 var(--page-hue)) 75%
      );
    box-shadow:
      color(display-p3 1 1 1 / 0.375) 0px 0.5px 0.5px 0px inset,
      color(display-p3 1 1 1 / 0.125) 0px 0px 0px 0.5px inset,
      color(display-p3 0 0.02 0.3) 0px 0px 1px 0px,
      color(display-p3 0 0.02 0.3 / 0.25) 0px 1px 1px 0px,
      color(display-p3 0 0.02 0.3 / 0.25) 0px 2px 8px 0px,
      color(display-p3 0 0.02 0.3 / 0.375) 0px 12px 12px 0px;
  }
</style>
