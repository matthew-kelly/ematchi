<script lang="ts">
	import { createEventDispatcher } from 'svelte';
	import Square from './Square.svelte';

	export let grid: string[];
	export let found: string[];

	const dispatch = createEventDispatcher();

	let a: number;
	let b: number;
	let reset_timeout: number;

	function handleClick(i: number) {
		if (a > -1 && b > -1) {
			clearTimeout(reset_timeout);
			a = i;
			b = -1;
		} else if (a > -1) {
			b = i;

			if (grid[a] === grid[b]) {
				// correct - remove from grid
				found = [...found, grid[a]];
				dispatch('found', { emoji: grid[a] });
			} else {
				// incorrect - reset after timeout
				reset_timeout = setTimeout(() => {
					a = b = -1;
				}, 1000);
			}
		} else {
			a = i;
		}
	}
</script>

{#each grid as emoji, i}
	<Square
		{emoji}
		selected={a === i || b === i}
		found={found.includes(emoji)}
		group={grid.indexOf(emoji) === i ? 'a' : 'b'}
		on:click={() => handleClick(i)}
	/>
{/each}
