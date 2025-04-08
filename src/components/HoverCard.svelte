<script>
  import Widget from "./Widget.svelte";

  let props = [
    {
      group: "Hover Effects",
      name: "scale",
      label: "Scale",
      value: 1.25,
      min: 1,
      max: 1.25,
      step: 0.005,
      unit: "",
    },
    {
      group: "Hover Effects",
      name: "tilt",
      label: "Tilt",
      value: 70,
      min: 1,
      max: 90,
      step: 1,
      unit: "deg",
    },
    {
      group: "Hover Effects",
      name: "shift",
      label: "Shift",
      value: 20,
      min: 1,
      max: 20,
      step: 1,
      unit: "px",
    },
  ];

  const handleMouseMove = ({ target, clientX, clientY }) => {
    const rect = target.getBoundingClientRect();

    const x = clientX - rect.left;
    const y = clientY - rect.top;

    const cursorX = `${x / rect.width}`;
    const cursorY = `${y / rect.height}`;

    requestAnimationFrame(() => {
      target.style.setProperty("--cursor-x", cursorX);
      target.style.setProperty("--cursor-y", cursorY);
    });
  };

  const resetStyles = ({ target }) => {
    requestAnimationFrame(() => {
      target.style.setProperty("--cursor-x", `0.5`);
      target.style.setProperty("--cursor-y", `0.5`);
    });
  };
</script>

<Widget {props}>
  <div class="row">
    <article
      class="card"
      on:mousemove={handleMouseMove}
      on:mouseleave={resetStyles}
    ></article>
    <article
      class="card"
      on:mousemove={handleMouseMove}
      on:mouseleave={resetStyles}
    ></article>
    <article
      class="card"
      on:mousemove={handleMouseMove}
      on:mouseleave={resetStyles}
    ></article>
  </div>
</Widget>

<style>
  .row {
    display: flex;
    gap: 2em;
  }

  .card {
    --blue-black: 0 0 0;
    --light-blue: 0.85 0.9 1;
    --light-pink: 1 0.75 1;

    --cursor-x: 0.5;
    --cursor-y: 0.5;

    --rotate-x: calc((var(--cursor-y) - 0.5) * var(--tilt));
    --rotate-y: calc((var(--cursor-x) - 0.5) * -1 * var(--tilt));
    --translate-x: calc((var(--cursor-x) - 0.5) * var(--shift));
    --translate-y: calc((var(--cursor-y) - 0.5) * var(--shift));
    --shadow-x: calc((0.5 - var(--cursor-x)) * 50px);
    --shadow-y: calc((0.5 - var(--cursor-y)) * 50px);
    --opacity-left: calc(var(--cursor-x) * 0.1875);
    --opacity-right: calc((1 - var(--cursor-x)) * 0.1875);

    width: 10rem;
    aspect-ratio: 1;
    background: linear-gradient(in oklch to bottom right, orange, purple);
    border-radius: 2em;
    transition: 200ms ease;
    transform: rotateX(var(--rotate-x)) rotateY(var(--rotate-y))
      translateX(var(--translate-x)) translateY(var(--translate-y));
  }
  .card:nth-of-type(1) {
    background: linear-gradient(in oklch to bottom right, orange, purple);
  }
  .card:nth-of-type(2) {
    background: linear-gradient(in oklch to bottom right, lime, blue);
  }
  .card:nth-of-type(3) {
    background: linear-gradient(in oklch to bottom right, hotpink, blue);
  }
  .card:hover {
    scale: var(--scale);
  }
  .card:active {
    scale: calc(var(--scale) * 0.95);
  }
  .card::before {
    content: "";
    position: absolute;
    top: 25%;
    left: 12.5%;
    width: 75%;
    height: 75%;
    align-content: center;
    border-radius: 21%;
    background: color(display-p3 var(--blue-black) / 0.375);
    box-shadow: var(--shadow-x) var(--shadow-y) 1em 1em
      color(display-p3 var(--blue-black) / 0.375);
    z-index: -1;
    transition: box-shadow 125ms ease;
    mix-blend-mode: screen;
    pointer-events: none;
  }
  .card::after {
    content: "";
    position: absolute;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    background: radial-gradient(
      circle at calc(var(--cursor-x) * 100%) calc(var(--cursor-y) * 100%),
      color(display-p3 var(--light-blue) / 0),
      transparent 75%
    );
    pointer-events: none;
    border-radius: 2em;
    overflow: hidden;
    mix-blend-mode: plus-lighter;
    transition: 200ms;
  }
  .card:hover::after {
    background: radial-gradient(
        circle at calc(var(--cursor-x) * 100%) calc(var(--cursor-y) * 100%),
        color(display-p3 var(--light-blue) / var(--opacity-left)),
        transparent 75%
      ),
      radial-gradient(
        circle at calc(var(--cursor-x) * 100%) calc(var(--cursor-y) * 100%),
        color(display-p3 var(--light-pink) / var(--opacity-right)),
        transparent 75%
      );
  }
</style>
