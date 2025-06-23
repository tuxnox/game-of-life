<script lang="ts">
	import { onMount } from 'svelte';

	let running = false;
	let interval: ReturnType<typeof setInterval> | null = null;

	function createMatrix(rows: number, columns: number) {
		let matrix: number[][] = [];
		for (let i = 0; i < rows; i++) {
			let r: number[] = [];
			for (let j = 0; j < columns; j++) {
				r.push(0);
			}
			matrix.push(r);
		}

		return matrix;
	}

	let rows = 10;
	let columns = 10;
	let matrix: number[][] = createMatrix(rows, columns);

	let population = 0;

	function initialState() {
		matrix = createMatrix(rows, columns);

		matrix[6][1] = 1;
		matrix[6][4] = 1;
		matrix[7][0] = 1;
		matrix[7][1] = 1;
		matrix[7][2] = 1;
		matrix[7][3] = 1;
		matrix[8][0] = 1;
		matrix[8][1] = 1;
		matrix[8][2] = 1;
		matrix[8][3] = 1;
		matrix[8][4] = 1;
	}

	function checkNeighboringCells(x: number, y: number) {
		/* 
            [[x-1][y-1]]    [[x-1][y]]  [[x-1][y+1]]
            [[x][y-1]]          [O]     [[x][y+1]]
            [[x+1][y-1]]    [[x+1][y]]  [[x+1][y+1]]
        */

		let alive = 0;

		for (let dx = -1; dx < 2; dx++) {
			for (let dy = -1; dy < 2; dy++) {
				let real_x = x + dx;
				let real_y = y + dy;

				if (real_x < 0 || real_x >= rows || real_y < 0 || real_y >= columns) {
					continue;
				}

				if (matrix[real_x][real_y] == 1) {
					alive++;
				}
			}
		}

		return alive;
	}

	function nextGeneration() {
		for (let i = 0; i < rows; i++) {
			let r: number[] = [];
			for (let j = 0; j < columns; j++) {
				let alive_cells = checkNeighboringCells(i, j);
				if (alive_cells < 2) {
					// Dies by underpopulation
					matrix[i][j] = 0;
				} else if (alive_cells >= 2 && alive_cells <= 3 && matrix[i][j] == 1) {
					matrix[i][j] = 1;
				} else if (alive_cells > 3) {
					matrix[i][j] = 0;
				} else if (alive_cells == 3) {
					matrix[i][j] = 1;
				}
			}
		}
	}

	initialState();

	function toggle() {
		if (running) {
			if (interval) clearInterval(interval);
			interval = null;
			running = false;
		} else {
			running = true;
			interval = setInterval(nextGeneration, 300);
		}
	}
</script>

<h1>Conway's Game of Life</h1>

<h2>Create Board</h2>
<!-- <input type="number" bind:value={rows} min="1" max="100" placeholder="Rows" /> -->
<!-- <input type="number" bind:value={columns} min="1" max="100" placeholder="Columns" /> -->

{#if matrix}
	<div class="board" style="grid-template-columns: repeat({columns}, 0fr);">
		{#each matrix as row, rowIndex}
			<div class="row">
				{#each row as cell, colIndex}
					<div class="cell" class:alive={cell === 1} class:dead={cell === 0}></div>
				{/each}
			</div>
		{/each}
	</div>
{/if}

<button on:click={() => initialState()}>Reset Board</button>

<button on:click={() => nextGeneration()}>Next Gen</button>

<button on:click={toggle}>{running ? 'Pause' : 'Continue'}</button>

<style>
	@import url('https://fonts.googleapis.com/css2?family=JetBrains+Mono:wght@400;700&display=swap');

	:root {
		--background-color: #0b0f1a;
		--grid-line: #192b3c;
		--alive-cell: #00aeef;
		--newborn-cell: #9f7aea;
		--dead-cell: #0b0f1a;
		--cell-glow: #ff5edc30;
		--cursor-hover: #ff5edc;
		--ui-foreground: #e0e0e0;
		--ui-accent: #00aeef;
	}

	:global(body) {
		background: var(--background-color);
		margin: 0;
		min-height: 100vh;
	}

	h1 {
		font-family: 'JetBrains Mono', monospace;
		text-align: center;
		margin-top: 2rem;
		font-size: 2.5rem;
		color: var(--ui-foreground);
	}

	h2 {
		font-family: 'JetBrains Mono', monospace;
		text-align: center;
		margin-top: 1rem;
		font-size: 1.5rem;
		color: var(--ui-foreground);
	}

	.board {
		display: grid;
		margin: 0 auto;
		max-width: 600px;
		gap: 0;
	}

	.row {
		display: contents;
	}

	.cell {
		width: 30px;
		height: 30px;
		background-color: var(--dead-cell);
		border: 1px solid var(--grid-line);
		position: relative;
		transition:
			background-color 0.3s,
			box-shadow 0.3s;
	}

	.cell.alive {
		background-color: var(--alive-cell);
		box-shadow: 0 0 10px var(--cell-glow);
	}

	.cell.dead {
		background-color: var(--dead-cell);
	}
</style>
