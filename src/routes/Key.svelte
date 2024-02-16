<script lang="ts">
	import { Item } from './logic';
	import KeyItem from './KeyItem.svelte';

	export let items: Array<Item>;
</script>

<div class="key">
	{#each items as item}
		<KeyItem style="color: {item.color};">{item.digit}</KeyItem>
	{/each}
</div>

<style>
	.key {
		display: grid;
		grid-template-columns: repeat(3, 1fr);
		grid-auto-rows: minmax(11px, auto);
		border-style: solid;
		background: white;
	}
</style>
