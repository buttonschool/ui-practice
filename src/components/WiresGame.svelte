<script>
  import Widget from "./Widget.svelte";

  // A sample prop array for styling via your Widget
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
    {
      name: "reflection-angle",
      label: "Reflection Angle",
      value: 30,
      min: 0,
      max: 90,
      step: 1,
      unit: "deg",
    },
    {
      name: "reflection-offset",
      label: "Reflection Offset",
      value: 60,
      min: 0,
      max: 100,
      step: 1,
      unit: "%",
    },
    {
      name: "reflection-opacity",
      label: "Reflection Opacity",
      value: 0.05,
      min: 0,
      max: 1,
      step: 0.01,
      unit: "",
    },
  ];

  // Board configuration
  let size = 6;
  let cellSize = 50;
  let gridGap = 2;

  // Define color pairs + endpoints; note that endpoints are not prefilled in the grid.
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
      color: "springgreen",
      endpoints: [
        [0, 1],
        [4, 2],
      ],
      path: [],
    },
    {
      color: "deepskyblue",
      endpoints: [
        [0, 3],
        [1, 1],
      ],
      path: [],
    },
    {
      color: "orangered",
      endpoints: [
        [0, 4],
        [4, 5],
      ],
      path: [],
    },
    {
      color: "gold",
      endpoints: [
        [1, 4],
        [5, 5], // bottom-right
      ],
      path: [],
    },
  ];

  // computedGrid is derived from the pairs' drawn paths.
  // It represents the cells that should be highlighted (filled with the pair's color).
  $: computedGrid = (() => {
    let g = Array(size)
      .fill(null)
      .map(() => Array(size).fill(null));
    pairs.forEach((pair) => {
      pair.path.forEach(([r, c]) => {
        g[r][c] = pair.color;
      });
    });
    return g;
  })();

  // Drag state
  let isDragging = false;
  let currentColor = null;
  let currentPath = [];
  let collidingCell = null; // For highlighting collisions

  // Helper: return true if (r, c) is an endpoint for the given pair.
  function isEndpoint(pair, r, c) {
    return pair.endpoints.some(([er, ec]) => er === r && ec === c);
  }

  // Helper: get a pair by its color.
  function getPair(color) {
    return pairs.find((p) => p.color === color);
  }

  // Helper: get a pair if (r, c) is an endpoint for it.
  function getPairAt(r, c) {
    return pairs.find((p) =>
      p.endpoints.some(([er, ec]) => er === r && ec === c)
    );
  }

  // Start drawing from a cell.
  // If the cell is an endpoint (even if not yet highlighted), use that pair.
  // Also, if the endpoint already has a drawn path, clicking it clears the path.
  function startDrawing(r, c) {
    let pair = getPairAt(r, c);
    if (!pair) return;
    let color = pair.color;
    // If clicked on an endpoint and a path already exists, clear it.
    if (isEndpoint(pair, r, c) && pair.path.length > 0) {
      clearPathForPair(pair);
      return;
    }
    isDragging = true;
    currentColor = color;
    currentPath = [[r, c]];
    // Set the starting cell into the pair's path.
    pair.path = [...currentPath];
    pairs = [...pairs];
    collidingCell = null;
  }

  // Clear a pair's drawn path.
  function clearPathForPair(pair) {
    pair.path = [];
    pairs = [...pairs];
  }

  // Handle pointer movement.
  // Updates the current pair's path (and thus the computedGrid) if movement is valid.
  function handlePointerMove(row, col) {
    if (!isDragging || !currentColor) return;
    const pair = getPair(currentColor);
    // Allow only adjacent (Manhattan distance 1) moves.
    if (currentPath.length > 0) {
      const [lastRow, lastCol] = currentPath[currentPath.length - 1];
      const dr = row - lastRow;
      const dc = col - lastCol;
      if (Math.abs(dr) + Math.abs(dc) !== 1) return;
    }
    // Backtracking: if the cell is already in the path, truncate the path there.
    const index = currentPath.findIndex(([r, c]) => r === row && c === col);
    if (index >= 0) {
      if (index < currentPath.length - 1) {
        currentPath = currentPath.slice(0, index + 1);
        pair.path = [...currentPath];
      }
      return;
    }
    // Collision check: if computedGrid already has a different color here and it's not an endpoint.
    if (
      computedGrid[row][col] &&
      computedGrid[row][col] !== currentColor &&
      !isEndpoint(pair, row, col)
    ) {
      collidingCell = [row, col];
      return;
    }
    // Otherwise, extend the path.
    currentPath.push([row, col]);
    pair.path = [...currentPath];
    pairs = [...pairs];
    if (collidingCell && collidingCell[0] === row && collidingCell[1] === col) {
      collidingCell = null;
    }
  }

  // End drawing.
  // If the final cell is a valid endpoint for the pair, the drawn line is kept;
  // otherwise, the path is cleared.
  function endDrawing() {
    if (!isDragging || !currentColor) return;
    const pair = getPair(currentColor);
    const [end1, end2] = pair.endpoints;
    const [lastRow, lastCol] = currentPath[currentPath.length - 1];
    const validEnd =
      (lastRow === end1[0] && lastCol === end1[1]) ||
      (lastRow === end2[0] && lastCol === end2[1]);
    if (!validEnd) {
      revertCurrentPath();
    } else {
      // If valid, ensure the final endpoint is included.
      if (!isEndpoint(pair, lastRow, lastCol)) {
        currentPath.push([lastRow, lastCol]);
        pair.path = [...currentPath];
        pairs = [...pairs];
      }
    }
    isDragging = false;
    currentColor = null;
    currentPath = [];
  }

  // Revert the current path (clear the drawn path for the current pair).
  function revertCurrentPath() {
    const pair = getPair(currentColor);
    currentPath = [];
    pair.path = [];
    pairs = [...pairs];
    collidingCell = null;
  }

  // Helper: Convert a path (an array of [r, c]) into an SVG polyline points string.
  function pointsFor(path) {
    return path
      .map(([r, c]) => {
        const x = c * (cellSize + gridGap) + cellSize / 2;
        const y = r * (cellSize + gridGap) + cellSize / 2;
        return `${x},${y}`;
      })
      .join(" ");
  }

  // Container-level pointer move: Compute cell coordinates from event position.
  let containerEl;
  function handleContainerPointerMove(event) {
    if (!isDragging) return;
    const boardEl = containerEl.querySelector(".board");
    const boardRect = boardEl.getBoundingClientRect();
    const relX = event.clientX - boardRect.left;
    const relY = event.clientY - boardRect.top;
    const col = Math.floor(relX / (cellSize + gridGap));
    const row = Math.floor(relY / (cellSize + gridGap));
    if (row >= 0 && row < size && col >= 0 && col < size) {
      handlePointerMove(row, col);
    }
  }

  // Keyboard support
  function handleKeyDown(r, c, event) {
    const key = event.key;
    if (key === "Enter" || key === " ") {
      event.preventDefault();
      if (!isDragging) {
        startDrawing(r, c);
      } else {
        handlePointerMove(r, c);
      }
    } else if (
      key === "ArrowUp" ||
      key === "ArrowDown" ||
      key === "ArrowLeft" ||
      key === "ArrowRight"
    ) {
      event.preventDefault();
      let newRow = r;
      let newCol = c;
      if (key === "ArrowUp") newRow = Math.max(r - 1, 0);
      if (key === "ArrowDown") newRow = Math.min(r + 1, size - 1);
      if (key === "ArrowLeft") newCol = Math.max(c - 1, 0);
      if (key === "ArrowRight") newCol = Math.min(c + 1, size - 1);
      const nextCell = document.getElementById(`cell-${newRow}-${newCol}`);
      if (nextCell) {
        nextCell.focus();
        if (isDragging) {
          handlePointerMove(newRow, newCol);
        }
      }
    } else if (key === "Escape") {
      if (isDragging) {
        revertCurrentPath();
        isDragging = false;
        currentColor = null;
        currentPath = [];
      }
    }
  }

  function handleKeyUp(r, c, event) {
    if ((event.key === "Enter" || event.key === " ") && isDragging) {
      endDrawing();
    }
  }

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
  <div
    class="container"
    bind:this={containerEl}
    role="application"
    on:pointermove={handleContainerPointerMove}
    on:pointerup|capture={() => {
      if (isDragging) endDrawing();
    }}
  >
    <div class="board" style={boardStyle}>
      {#each computedGrid as row, r}
        {#each row as cellColor, c}
          <button
            id={`cell-${r}-${c}`}
            class="cell {collidingCell &&
            collidingCell[0] === r &&
            collidingCell[1] === c
              ? 'collision'
              : ''}"
            style="width: {cellSize}px; height: {cellSize}px; --color: {cellColor ||
              'transparent'}"
            on:pointerdown|preventDefault={() => startDrawing(r, c)}
            on:keydown={(e) => handleKeyDown(r, c, e)}
            on:keyup={(e) => handleKeyUp(r, c, e)}
          ></button>
        {/each}
      {/each}
    </div>

    <svg class="overlay" style={overlayStyle}>
      {#each pairs as pair}
        {#if pair.path && pair.path.length > 1}
          <polyline
            stroke={pair.color}
            stroke-width="8"
            fill="none"
            stroke-linecap="round"
            stroke-linejoin="round"
            points={pointsFor(pair.path)}
          />
        {/if}
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
    --reflection-color: rgba(255, 255, 255, var(--reflection-opacity));
    content: "";
    position: absolute;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    background: linear-gradient(
      var(--reflection-angle),
      transparent var(--reflection-offset),
      var(--reflection-color) var(--reflection-offset)
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
    -webkit-user-drag: none;
    user-drag: none;
  }
  .cell.collision {
    outline: 2px solid red;
  }
  .overlay {
    position: absolute;
    top: calc(var(--frame-width) + 0.25rem);
    left: calc(var(--frame-width) + 0.25rem);
    pointer-events: none;
    z-index: 10;
  }
</style>
