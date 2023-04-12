<script lang="ts">
	import { createEventDispatcher } from 'svelte';
	import Countdown from './Countdown.svelte';
	import Found from './Found.svelte';
	import Grid from './Grid.svelte';
	import { shuffle } from './utils';
	import type { Level } from './levels';

	const dispatch = createEventDispatcher();

	let size: number;
	let grid: string[] = [];
	let found: string[] = [];
	let remaining: number;
	let duration: number;
	let playing: boolean;

	export function start(level: Level) {
		size = level.size;
		remaining = duration = level.duration;

		const copy = level.emojis.slice();
		const pairs: string[] = [];

		// pick emojis for game
		for (let i = 0; i < size ** 2 / 2; i++) {
			const index = Math.floor(Math.random() * copy.length);
			const emoji = copy[index];
			copy.splice(index, 1);
			pairs.push(emoji);
		}

		// duplicate picked emojis
		grid = shuffle([...pairs, ...pairs]);
		found = [];

		resume();
	}

	export function resume() {
		playing = true;
		countdown();
		dispatch('play');
	}

	function countdown() {
		const start = Date.now();
		let remaining_at_start = remaining;

		function loop() {
			if (!playing) return;

			requestAnimationFrame(loop);

			remaining = remaining_at_start - (Date.now() - start);
			if (remaining <= 0) {
				playing = false;
				dispatch('lose');
			}
		}

		loop();
	}
</script>

<div class="game" style="--size: {size}">
	<div class="info">
		<Countdown
			{remaining}
			{duration}
			on:click={() => {
				playing = false;
				dispatch('pause');
			}}
		/>
	</div>
	<div class="grid">
		{#key grid}
			<Grid
				{grid}
				{found}
				on:found={(e) => {
					found = [...found, e.detail.emoji];

					if (found.length === (size * size) / 2) {
						playing = false;
						setTimeout(() => {
							playing = false;
							dispatch('win');
						}, 1000);
					}
				}}
			/>
		{/key}
	</div>
	<div class="info">
		<Found {found} />
	</div>
</div>

<style>
	.game {
		display: flex;
		flex-direction: column;
		justify-content: center;
		align-items: center;
		font-size: min(1vmin, 0.5em);
		gap: 2em;
		height: 100%;
	}

	.info {
		display: flex;
		flex-direction: column;
		justify-content: center;
		align-items: center;
		width: 80em;
		height: 10em;
	}

	.grid {
		display: grid;
		grid-template-columns: repeat(var(--size), 1fr);
		grid-template-rows: repeat(var(--size), 1fr);
		grid-gap: 1em;
		width: 80em;
		aspect-ratio: 1;
		perspective: 100vw;
		z-index: 2;
	}

	@media (min-aspect-ratio: 1/1) {
		.game {
			flex-direction: row-reverse;
		}
		.info {
			width: 10em;
			height: 80em;
		}
	}
</style>
