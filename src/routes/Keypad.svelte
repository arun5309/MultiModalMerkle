<script lang="ts">
	import Key from './Key.svelte';
	import KeyItem from './KeyItem.svelte';
	import { Item, Mode, get_ith_color } from './logic';

	export let keys: Array<Array<Item>>;
	export let mode: Mode = Mode.DISPLAY;

	export let bkspc_handler: () => void;
	export let enter_handler: () => void;
	export let press_handler: (i: number) => void;
</script>

<div class="keypad">
	{#each keys as key, i}
		{#if mode === Mode.DISPLAY}
			{#if i === 9}
				<KeyItem style="border-style: solid;" on:click={() => bkspc_handler()}>Backspace</KeyItem>
				<Key items={key} on:click={() => press_handler(i)} />
				<KeyItem style="border-style: solid;" on:click={() => enter_handler()}>Enter</KeyItem>
			{:else}
				<Key items={key} on:click={() => press_handler(i)} />
			{/if}
		{:else if mode === Mode.POSITION_ENTRY}
			{#if i === 9}
				<KeyItem style="border-style: solid;" on:click={() => bkspc_handler()}>Backspace</KeyItem>
				<KeyItem style="border-style: solid;" on:click={() => press_handler(i)} />
				<KeyItem style="border-style: solid;" on:click={() => enter_handler()}>Enter</KeyItem>
			{:else}
				<KeyItem style="border-style: solid;" on:click={() => press_handler(i)} />
			{/if}
		{:else if i === 9}
			<KeyItem style="border-style: solid;" on:click={() => bkspc_handler()}>Backspace</KeyItem>
			<KeyItem
				style="border-style: solid; background: {get_ith_color(i)}"
				on:click={() => press_handler(i)}
			/>
			<KeyItem style="border-style: solid;" on:click={() => enter_handler()}>Enter</KeyItem>
		{:else}
			<KeyItem
				style="border-style: solid; background: {get_ith_color(i)}"
				on:click={() => press_handler(i)}
			/>
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
