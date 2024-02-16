<script lang="ts">
	import Keypad from './Keypad.svelte';
	import { make_random_puzzles } from './logic';

	let value = make_random_puzzles();
</script>

<svelte:head>
	<title>MMM PIN Entry</title>
	<meta name="description" content="MMM PIN Entry" />
</svelte:head>

<h1>Multi-Modal Merkle PIN Entry</h1>

<Keypad keys={value}/>


<style>
	h1 {
		width: 100%;
	}
</style>
