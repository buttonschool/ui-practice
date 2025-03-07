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

  // Define color pairs + endpoints
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

  // Create a size×size grid (null if empty, else color string)
  let grid = Array(size)
    .fill(null)
    .map(() => Array(size).fill(null));

  // Remove pre-filling endpoints in grid:
  // (Endpoints will be shown via the SVG overlay until a valid line is drawn)

  // Drag state
  let isDragging = false;
  let currentColor = null;
  let currentPath = [];
  let collidingCell = null; // highlight if collision with another color

  // New helper: return true if (r, c) is an endpoint for the pair
  function isEndpoint(pair, r, c) {
    return pair.endpoints.some(([er, ec]) => er === r && ec === c);
  }

  // Helper: find a pair by color
  function getPair(color) {
    return pairs.find((p) => p.color === color);
  }

  // In startDrawing, update the grid immediately for the starting cell
  function startDrawing(r, c) {
    let color = grid[r][c];
    let pair = null;
    if (!color) {
      // If grid is empty here, check if (r, c) is an endpoint for any pair.
      pair = pairs.find((p) =>
        p.endpoints.some(([er, ec]) => er === r && ec === c)
      );
      if (pair) {
        color = pair.color;
      } else {
        return;
      }
    } else {
      pair = getPair(color);
      if (!pair) return;
    }
    // If the clicked cell is an endpoint and the pair already has a drawn path, clear it.
    if (isEndpoint(pair, r, c) && pair.path && pair.path.length > 1) {
      clearPathForPair(pair);
      return;
    }
    isDragging = true;
    currentColor = color;
    currentPath = [[r, c]];
    // Highlight the starting cell immediately
    grid[r][c] = currentColor;
    grid = grid.map((r) => [...r]);
    pair.path = [...currentPath];
    collidingCell = null;
    pairs = [...pairs]; // Force reactivity
  }

  // New helper to clear a pair's drawn path from the grid (except endpoints)
  function clearPathForPair(pair) {
    pair.path.forEach(([r, c]) => {
      if (!isEndpoint(pair, r, c)) {
        grid[r][c] = null;
      }
    });
    pair.path = [];
    grid = grid.map((r) => [...r]);
    pairs = pairs.map((p) => (p.color === pair.color ? { ...p, path: [] } : p));
  }

  // In handlePointerMove, update the grid for the new cell unconditionally
  function handlePointerMove(row, col) {
    if (!isDragging || !currentColor) return;
    const pair = getPair(currentColor);

    // Ensure movement is only to adjacent cells (no diagonal or jump moves)
    if (currentPath.length > 0) {
      const [lastRow, lastCol] = currentPath[currentPath.length - 1];
      const dr = row - lastRow;
      const dc = col - lastCol;
      // Only allow moves where Manhattan distance equals 1
      if (Math.abs(dr) + Math.abs(dc) !== 1) {
        return; // Ignore non-adjacent moves
      }
    }

    // Check if the cell is already in our current path (for backtracking)
    const index = currentPath.findIndex(([r, c]) => r === row && c === col);
    if (index >= 0) {
      if (index < currentPath.length - 1) {
        currentPath = currentPath.slice(0, index + 1);
        pair.path = [...currentPath];
        // Clear any grid cells that are no longer in the path (but keep endpoints intact)
        for (let i = 0; i < size; i++) {
          for (let j = 0; j < size; j++) {
            if (grid[i][j] === currentColor) {
              const inPath = currentPath.some(
                ([pr, pc]) => pr === i && pc === j
              );
              if (!inPath && !isEndpoint(pair, i, j)) {
                grid[i][j] = null;
              }
            }
          }
        }
        grid = grid.map((r) => [...r]);
        pairs = pairs.map((p) =>
          p.color === currentColor ? { ...p, path: [...currentPath] } : p
        );
      }
      return;
    }

    // If the cell is occupied by a different color (and it's not an endpoint for our pair), highlight collision and do nothing
    if (
      grid[row][col] &&
      grid[row][col] !== currentColor &&
      !isEndpoint(pair, row, col)
    ) {
      collidingCell = [row, col];
      return;
    }

    // Otherwise, extend the path.
    // Update the grid for the new cell regardless of whether it's an endpoint.
    grid[row][col] = currentColor;
    grid = grid.map((r) => [...r]);
    currentPath.push([row, col]);
    pair.path = [...currentPath];
    pairs = [...pairs];
    if (collidingCell && collidingCell[0] === row && collidingCell[1] === col) {
      collidingCell = null;
    }
  }

  // End drawing (pointerup or Enter/Space release)
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
      // For a valid completed line, update grid for the endpoint as well.
      grid[lastRow][lastCol] = currentColor;
      grid = grid.map((r) => [...r]);
    }
    isDragging = false;
    currentColor = null;
    currentPath = [];
  }

  // Revert the current path (clear non-endpoint cells)
  function revertCurrentPath() {
    const pair = getPair(currentColor);
    currentPath.forEach(([r, c]) => {
      if (!isEndpoint(pair, r, c)) {
        grid[r][c] = null;
      }
    });
    grid = grid.map((r) => [...r]);
    pair.path = [];
    pairs = pairs.map((p) =>
      p.color === currentColor ? { ...p, path: [] } : p
    );
    collidingCell = null;
  }

  // Convert path to an SVG polyline string
  function pointsFor(path) {
    return path
      .map(([r, c]) => {
        const x = c * (cellSize + gridGap) + cellSize / 2;
        const y = r * (cellSize + gridGap) + cellSize / 2;
        return `${x},${y}`;
      })
      .join(" ");
  }

  // Container-level pointer move: figure out cell from event coords
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
  <!-- The top-level container with pointer events -->
  <div
    class="container"
    bind:this={containerEl}
    role="application"
    on:pointermove={handleContainerPointerMove}
    on:pointerup|capture={() => {
      if (isDragging) endDrawing();
    }}
  >
    <!-- The grid -->
    <div class="board" style={boardStyle}>
      {#each grid as row, r}
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

    <!-- The SVG overlay for wires and endpoints -->
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

  .overlay {
    position: absolute;
    top: calc(var(--frame-width) + 0.25rem);
    left: calc(var(--frame-width) + 0.25rem);
    pointer-events: none;
    z-index: 10;
  }
</style>
