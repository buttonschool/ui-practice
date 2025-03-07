<script>
  import Widget from "./Widget.svelte";

  let props = [
    {
      name: "frame-width",
      label: "Frame Width",
      value: 2,
      min: 1,
      max: 6,
      step: 0.125,
      unit: "rem",
    },
  ];

  // Dimensions & styling
  let size = 6;
  let cellSize = 50;
  let gridGap = 2;

  // Your chosen color pairs + endpoints
  let pairs = [
    {
      color: "hotpink",
      endpoints: [
        [0, 0], // top-left
        [5, 0], // bottom-left
      ],
      path: [],
    },
    {
      color: "aquamarine",
      endpoints: [
        [0, 1],
        [4, 2],
      ],
      path: [],
    },
    {
      color: "dodgerblue",
      endpoints: [
        [0, 3],
        [1, 1],
      ],
      path: [],
    },
    {
      color: "red",
      endpoints: [
        [0, 4],
        [4, 5],
      ],
      path: [],
    },
    {
      color: "orange",
      endpoints: [
        [1, 4],
        [5, 5], // bottom-right
      ],
      path: [],
    },
  ];

  // Create a 2D array (size×size), initially null in every cell
  let grid = Array(size)
    .fill(null)
    .map(() => Array(size).fill(null));

  // Place each pair’s endpoints onto the grid
  pairs.forEach((pair) => {
    pair.endpoints.forEach(([r, c]) => {
      grid[r][c] = pair.color;
    });
  });

  // State for dragging
  let isDragging = false;
  let currentColor = null;
  let currentPath = [];

  // Helper to find a pair object by color
  function getPair(color) {
    return pairs.find((p) => p.color === color);
  }

  // Mouse down on a cell that has a color => start drawing that color
  function handleMouseDown(row, col) {
    console.log("MouseDown on cell", row, col);
    const color = grid[row][col];
    if (!color) return;

    const pair = getPair(color);
    if (!pair) return;

    isDragging = true;
    currentColor = color;
    currentPath = [[row, col]];
  }

  // Mouse enters a new cell while dragging => continue drawing path
  function handleMouseEnter(row, col) {
    console.log("MouseEnter cell", row, col);
    if (!isDragging || !currentColor) return;

    // Collision detection: if occupied by a different color, revert
    if (grid[row][col] && grid[row][col] !== currentColor) {
      revertCurrentPath();
      return;
    }

    // Mark this cell with the current color
    grid[row][col] = currentColor;
    // Force reactivity by reassigning grid
    grid = grid.map((r) => [...r]);

    currentPath.push([row, col]);
    const pair = getPair(currentColor);
    pair.path = [...currentPath];
    console.log("Current path for", currentColor, pair.path);

    // Force reactivity by reassigning pairs
    pairs = [...pairs];
  }

  // Mouse up => check if we ended on a valid endpoint
  function handleMouseUp() {
    console.log("MouseUp");
    if (!isDragging || !currentColor) return;

    const pair = getPair(currentColor);
    const [end1, end2] = pair.endpoints;
    const [lastRow, lastCol] = currentPath[currentPath.length - 1];

    // Must end on the matching endpoint or revert
    const validEnd =
      (lastRow === end1[0] && lastCol === end1[1]) ||
      (lastRow === end2[0] && lastCol === end2[1]);

    if (!validEnd) {
      revertCurrentPath();
    }

    isDragging = false;
    currentColor = null;
    currentPath = [];
  }

  // Revert any drawn cells that are not endpoints
  function revertCurrentPath() {
    console.log("Reverting path:", currentPath);
    const pair = getPair(currentColor);
    currentPath.forEach(([r, c]) => {
      const isEndpoint = pair.endpoints.some(
        ([er, ec]) => er === r && ec === c
      );
      if (!isEndpoint) {
        grid[r][c] = null;
      }
    });
    grid = grid.map((r) => [...r]);
    pair.path = [];
  }

  // Convert path ([ [r,c], [r,c], ... ]) into an SVG polyline string
  function pointsFor(path) {
    return path
      .map(([r, c]) => {
        // Each cell is offset by (cellSize + gridGap) horizontally & vertically
        const x = c * (cellSize + gridGap) + cellSize / 2;
        const y = r * (cellSize + gridGap) + cellSize / 2;
        return `${x},${y}`;
      })
      .join(" ");
  }

  // Dynamic inline styles
  $: boardStyle = `
    grid-template-columns: repeat(${size}, ${cellSize}px);
    grid-template-rows: repeat(${size}, ${cellSize}px);
    gap: ${gridGap}px;
  `;
  $: overlayStyle = `
    width: ${size * (cellSize + gridGap)}px;
    height: ${size * (cellSize + gridGap)}px;
  `;
</script>

<Widget {props}>
  <!-- svelte-ignore a11y-no-noninteractive-element-interactions -->
  <div class="container" role="application" on:mouseup|capture={handleMouseUp}>
    <!-- The clickable grid -->
    <div class="board" style={boardStyle}>
      {#each grid as row, r}
        {#each row as cellColor, c}
          <button
            class="cell"
            style="width: {cellSize}px; height: {cellSize}px; --color: {cellColor ||
              'transparent'}"
            on:mousedown|preventDefault={() => handleMouseDown(r, c)}
            on:mouseenter={() => handleMouseEnter(r, c)}
          ></button>
        {/each}
      {/each}
    </div>

    <!-- SVG overlay for drawing lines and showing endpoints -->
    <svg class="overlay" style={overlayStyle}>
      {#each pairs as pair}
        {#if pair.path && pair.path.length > 1}
          <!-- White underlay for contrast -->
          <!-- <polyline
            stroke="white"
            stroke-width="14"
            fill="none"
            stroke-linecap="round"
            stroke-linejoin="round"
            points={pointsFor(pair.path)}
          /> -->
          <!-- Colored wire on top -->
          <polyline
            stroke={pair.color}
            stroke-width="8"
            fill="none"
            stroke-linecap="round"
            stroke-linejoin="round"
            points={pointsFor(pair.path)}
          />
        {/if}

        <!-- Draw endpoints as circles so they're visible -->
        {#each pair.endpoints as [r, c]}
          <circle
            cx={c * (cellSize + gridGap) + cellSize / 2}
            cy={r * (cellSize + gridGap) + cellSize / 2}
            r={cellSize * 0.2}
            fill="black"
            stroke={pair.color}
            stroke-width="8"
          />
        {/each}
      {/each}
    </svg>
  </div>
</Widget>

<style>
  .container {
    position: relative;
    width: max-content;
    user-select: none;
    background-color: rgb(237, 236, 223);
    padding: var(--frame-width);
    border-radius: calc(var(--frame-width) + 0.75rem);
    box-shadow: 0 1rem 0 0 rgb(211, 209, 201);
    overflow: hidden;
  }
  .container::before {
    content: "";
    position: absolute;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    background: linear-gradient(
      30deg,
      transparent 60%,
      rgba(255, 255, 255, 0.05) 60%
    );
    pointer-events: none;
  }

  .board {
    display: grid;
    background-color: #111;
    padding: 0.25rem;
    border-radius: 0.75rem;
  }

  .cell {
    border-radius: 0.5rem;
    border: none;
    background-color: #333;
    background-color: color-mix(in oklch, #333 80%, var(--color));
    /* Ensures no browser-native drag starts if you move quickly */
    -webkit-user-drag: none;
  }

  .overlay {
    position: absolute;
    top: calc(var(--frame-width) + 0.25rem);
    left: calc(var(--frame-width) + 0.25rem);
    pointer-events: none; /* Let clicks pass through to .board */
    z-index: 10;
  }
</style>
