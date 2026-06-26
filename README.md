<img width="866" height="679" alt="image" src="https://github.com/user-attachments/assets/cfd492cd-ce1f-4325-b2db-ba4727a53167" />
<div align="center">

# Pathfinder

### Watch search algorithms think.

An interactive pathfinding visualizer that renders **BFS**, **DFS**, **Dijkstra**, and **A\*** exploring a grid in real time — then threads the shortest path through it. Draw walls, drag the endpoints, generate mazes, and *see* why a heuristic-guided search reaches the goal after exploring a fraction of the cells.

Built as a **single, dependency-free HTML file**. No build step. No framework. Just open it.

<br>

![License](https://img.shields.io/badge/license-MIT-3b82f6.svg)
![JavaScript](https://img.shields.io/badge/JavaScript-Vanilla-f7df1e?logo=javascript&logoColor=black)
![HTML5](https://img.shields.io/badge/HTML5-single--file-e34f26?logo=html5&logoColor=white)
![Dependencies](https://img.shields.io/badge/dependencies-0-22c55e.svg)
![PRs Welcome](https://img.shields.io/badge/PRs-welcome-6366f1.svg)

<br>

<!-- Replace this placeholder with a screen recording of the app in action. -->
<!-- A short GIF showing A* vs BFS exploring the grid lands hardest here.   -->
![Pathfinder demo](docs/demo.gif)

</div>

---

## Table of Contents

- [Overview](#overview)
- [Features](#features)
- [Algorithms](#algorithms)
- [Tech Stack](#tech-stack)
- [Getting Started](#getting-started)
- [Usage](#usage)
- [How It Works](#how-it-works)
- [Configuration](#configuration)
- [Accessibility](#accessibility)
- [Project Structure](#project-structure)
- [Limitations & Trade-offs](#limitations--trade-offs)
- [Roadmap](#roadmap)
- [Contributing](#contributing)
- [License](#license)
- [Author](#author)

---

## Overview

Pathfinder turns classic graph-traversal algorithms into something you can watch and play with. Every cell on the grid is a node; adjacent cells share an edge of weight 1. As an algorithm runs, the search frontier *blooms* outward in discovery order, and once the target is reached the shortest path is reconstructed and animated back in.

The point of the project is **intuition through motion**. Run A\*, then run BFS on the same grid and compare the **Explored** counter:

| Algorithm | Cells explored (open grid) | Path length |
| :-- | :--: | :--: |
| BFS | ~66 | 8 |
| A\* | ~9 | 8 |

Same optimal path, a fraction of the work — that single comparison is the whole case for heuristic search, made visible.

---

## Features

- **Four algorithms** — Breadth-First Search, Depth-First Search, Dijkstra, and A\* with a Manhattan heuristic.
- **Live wall editing** — click and drag to paint walls; drag over existing walls to erase.
- **Draggable endpoints** — move the source and target anywhere. When a result is on screen, the shortest path **re-solves instantly** as you drag the node.
- **Maze generation** — recursive-division mazes that are guaranteed solvable.
- **Animated search frontier** — explored cells reveal in true discovery order, so the wave shape tells you *how* each algorithm thinks.
- **Adjustable speed** — Slow, Normal, and Fast playback.
- **Live telemetry** — current algorithm, cells explored, path length, and status (Idle / Searching / Path found / No path).
- **Touch and pen support** — built on the Pointer Events API, so it works on desktop and mobile.
- **Zero dependencies** — one HTML file, no install, no bundler.

---

## Algorithms

All four operate on an unweighted grid with 4-directional movement (up, down, left, right). `V` is the number of cells (`ROWS × COLS`) and `E` is the number of edges.

| Algorithm | Strategy | Guarantees shortest path? | Time | Space |
| :-- | :-- | :--: | :--: | :--: |
| **BFS** | Expands uniformly in rings from the source. | Yes (unweighted) | `O(V + E)` | `O(V)` |
| **DFS** | Dives as deep as possible before backtracking. | No | `O(V + E)` | `O(V)` |
| **Dijkstra** | Always expands the lowest-cost frontier node. | Yes | `O(V²)` *(see note)* | `O(V)` |
| **A\*** | Dijkstra plus a Manhattan heuristic that steers toward the target. | Yes (admissible heuristic) | `O(V²)` *(see note)* | `O(V)` |

> **Note on Dijkstra/A\*:** the priority frontier is implemented as a linear-scan minimum selection rather than a binary heap. This keeps the code compact and is comfortably fast at this grid size (~989 cells), but it is `O(V²)` in the worst case. A binary heap would bring it to `O((V + E) log V)` for larger grids — see [Roadmap](#roadmap).

Because every edge costs 1, **Dijkstra is functionally equivalent to BFS here**. Add weighted terrain (on the roadmap) and the two visibly diverge.

---

## Tech Stack

| Layer | Choice | Why |
| :-- | :-- | :-- |
| Markup | HTML5 | Single self-contained document. |
| Styling | Modern CSS (Grid, custom properties, keyframes) | Responsive square-cell grid, themed via CSS variables, GPU-friendly animations. |
| Logic | Vanilla JavaScript (ES2015+) | No framework overhead; direct, imperative DOM updates for smooth animation. |
| Input | Pointer Events API | Unifies mouse, touch, and pen with a single code path. |
| Type | Space Grotesk + JetBrains Mono (Google Fonts) | Loaded via CDN with system-font fallbacks. |

---

## Getting Started

No installation or build is required.

```bash
# 1. Clone the repository
git clone https://github.com/<your-username>/pathfinder.git
cd pathfinder
```

```bash
# 2a. Just open it — no server needed
open pathfinder.html          # macOS
xdg-open pathfinder.html      # Linux
start pathfinder.html         # Windows
# ...or simply double-click the file in your file explorer.
```

```bash
# 2b. Or serve it locally (recommended for consistent web-font loading)
python3 -m http.server 8000
# then visit http://localhost:8000/pathfinder.html

# alternatively
npx serve .
```

---

## Usage

| Action | How |
| :-- | :-- |
| Build a wall | Click and drag across empty cells. |
| Erase a wall | Click and drag starting on an existing wall. |
| Move the source / target | Drag the green source or the rose target node. |
| Run the search | Click **Visualize** (or press `Enter` / `Space`). |
| Generate a maze | Click **Generate maze**. |
| Change playback speed | Use the **Slow / Normal / Fast** control. |
| Clear the search overlay | Click **Clear path** (walls are kept). |
| Remove all walls | Click **Clear walls**. |
| Restore the starting layout | Click **Reset**. |

While a result is displayed, dragging the source or target re-solves and redraws the shortest path live, without replaying the full search animation.

---

## How It Works

The visualizer separates **computation** from **animation**, which keeps playback smooth and makes cancellation trivial.

**1. Grid model.** The board is a 2D array of node objects holding wall state. A parallel 2D array caches the corresponding DOM cells, so updates mutate class names directly — there is no virtual DOM or re-render cost per frame.

**2. Search core.** Each algorithm is a pure function that returns two things: the list of cells in the order they were explored, and the reconstructed path (via parent pointers). The path comes back empty when the target is unreachable.

**3. Animation pipeline.** The chosen algorithm runs synchronously to completion, then the explored cells are revealed with a staggered `setTimeout`. A monotonically increasing run token guards every scheduled callback, so any reset, clear, or new run cancels the in-flight animation cleanly — stale callbacks simply return:

```js
const myId = ++runId;              // claim this run
visited.forEach((cell, i) => {
  setTimeout(() => {
    if (myId !== runId) return;    // a newer run/reset superseded us — bail out
    paintExplored(cell);
  }, i * stepDelay);
});
```

**4. Live re-solve.** Dragging an endpoint after a result triggers an instant, path-only recompute. Search-frontier animations are suppressed (via an `instant` mode) so the path tracks the node in real time instead of replaying the whole wave.

**5. Maze generation.** Mazes use **recursive division**: the chamber is split by a wall with a single gap, then each half is divided recursively. After generation, a reachability check (a quick BFS) runs; in the rare event the source and target are disconnected, an L-shaped corridor is carved to guarantee a solvable maze.

**6. Input handling.** Pointer Events unify mouse, touch, and pen. On `pointerdown` the grid captures the pointer; subsequent moves use `document.elementFromPoint` to resolve which cell is under the cursor, which makes drag-painting reliable even across rapid movement.

---

## Configuration

The behavior is driven by a small set of constants at the top of the script in `pathfinder.html`. Adjust and reload.

| Constant | Default | Purpose |
| :-- | :-- | :-- |
| `ROWS` | `23` | Number of grid rows. |
| `COLS` | `43` | Number of grid columns. |
| `DEFAULT_START` | `{ r: 11, c: 9 }` | Initial source position. |
| `DEFAULT_END` | `{ r: 11, c: 33 }` | Initial target position. |
| `SPEED` | `{ slow: 42, normal: 16, fast: 5 }` | Delay in milliseconds per explored cell. |
| `PATH_DELAY` | `26` | Delay in milliseconds per path cell. |

> Odd values for `ROWS` and `COLS` work best with recursive-division maze generation.

---

## Accessibility

- **Keyboard support** — `Enter` or `Space` runs the search.
- **Visible focus** — all interactive controls have `:focus-visible` outlines.
- **Reduced motion** — honors `prefers-reduced-motion`; the bloom, pop, and pulse animations are disabled for users who request it.
- **Responsive** — the grid scales from desktop down to mobile; touch input is supported.
- **Live regions** — the telemetry strip uses `aria-live` so status changes are announced.

---

## Project Structure

```
pathfinder/
├── index.html     # The entire application: HTML, CSS, and JS in one file
└── README.md
```

---

## Limitations & Trade-offs

These are deliberate scoping choices, documented for transparency:

- **Unweighted edges.** Every step costs 1, so Dijkstra and BFS behave identically. Weighted terrain is the most impactful next feature.
- **4-directional movement only.** No diagonals. The Manhattan heuristic is admissible for this movement model; switching to diagonals would require an octile/Chebyshev heuristic.
- **Linear-scan priority frontier.** Simple and fast enough for this grid; not optimal asymptotically (see [Algorithms](#algorithms)).
- **CDN web fonts.** Typography loads from Google Fonts and falls back to system fonts when offline.

---

## Roadmap

- [ ] **Weighted cells** — paintable terrain costs so Dijkstra and A\* diverge from BFS.
- [ ] **Diagonal movement** — with an octile heuristic for A\*.
- [ ] **Binary-heap priority queue** — `O((V + E) log V)` to scale to large grids.
- [ ] **Side-by-side race** — two algorithms on twin grids simultaneously.
- [ ] **Additional maze generators** — recursive backtracker, Prim's, randomized scatter.
- [ ] **Shareable boards** — encode the grid in the URL.
- [ ] **Bidirectional and Greedy Best-First search.**

---

## Contributing

Contributions are welcome.

1. Fork the repository.
2. Create a feature branch: `git checkout -b feature/weighted-cells`.
3. Commit your changes: `git commit -m "Add weighted cells"`.
4. Push the branch: `git push origin feature/weighted-cells`.
5. Open a pull request.

For substantial changes, please open an issue first to discuss the direction.

---

## License

Released under the [MIT License](LICENSE). You are free to use, modify, and distribute it.

---

## Author

**Kunj**

- GitHub: [@your-handle](https://github.com/your-handle)
- LinkedIn: [your profile](https://linkedin.com/in/your-profile)

If this project helped you understand pathfinding — or just made graph traversal a little more fun — consider giving it a star.
