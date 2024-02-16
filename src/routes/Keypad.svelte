<script lang="ts">
	import Key from './Key.svelte';
    import KeyItem from './KeyItem.svelte';
	import { Item } from './logic';

	export let keys: Array<Array<Item>>;
</script>

<div class="keypad">
	{#each keys as key, i}
		{#if i === 9}
			<KeyItem style="border-style: solid;">Backspace</KeyItem>
			<Key items={key} />
			<KeyItem style="border-style: solid;">Enter</KeyItem>
		{:else}
			<Key items={key} />
		{/if}
	{/each}
</div>

<style>
	.keypad {
		display: grid;
		grid-template-columns: repeat(3, 1fr);
		grid-auto-rows: minmax(100px, auto);
		border-style: solid;
		background: white;
	}
</style>
