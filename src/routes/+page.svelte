<script lang="ts">
	import Keypad from './Keypad.svelte';
	import { make_random_puzzles, Mode, GameState, NCOLORS, get_ith_color } from './logic';

	let game_state = GameState.START;
	let keys = make_random_puzzles();
	let mode = Mode.DISPLAY;

	let value = '';
	$: round = value.length + 1;

	let show_pin = false;
	let cur_pos: string = '';
	let cur_color: string = '';

	function to_color_num(color: string): number {
		if (!color) {
			return NCOLORS + 1;
		}
		return Number(color);
	}

	function bkspc_handler() {
		if (value) {
			value = value.slice(0, value.length - 1);
		}
	}

	function enter_handler() {
		switch (mode) {
			case Mode.DISPLAY:
				mode = Mode.POSITION_ENTRY;
				break;
			case Mode.POSITION_ENTRY:
				if (!cur_pos) {
					break;
				}
				mode = Mode.COLOR_ENTRY;
				break;
			case Mode.COLOR_ENTRY:
				if (!cur_color) {
					break;
				}
				value = value + String(keys[Number(cur_pos)][to_color_num(cur_color)].digit);
				if (value.length == 4) {
					game_state = GameState.FINISH;
					break;
				}
				keys = make_random_puzzles();
				mode = Mode.DISPLAY;
				cur_pos = '';
				cur_color = '';
				break;
		}
	}

	function press_handler(i: number) {
		switch (mode) {
			case Mode.DISPLAY:
				break;
			case Mode.POSITION_ENTRY:
				cur_pos = String(i);
				break;
			case Mode.COLOR_ENTRY:
				if (i < NCOLORS) cur_color = String(i);
				break;
		}
	}
</script>

<svelte:head>
	<title>MMM PIN Entry</title>
	<meta name="description" content="MMM PIN Entry" />
</svelte:head>

<h1>Multi-Modal Merkle PIN Entry</h1>

{#if game_state === GameState.START}
	<button on:click={() => (game_state = GameState.PROGRESS)}>Start Game</button>
{:else if game_state === GameState.PROGRESS}
	<h2>Round: {round}</h2>
	<br />

	<div>
		Cur Pos: {cur_pos}
		Cur Color:
		<div style="background: {get_ith_color(to_color_num(cur_color))};" class="colorbox"></div>
	</div>
	<br />

	<!-- Useful for debugging! 
		<div>PIN Cur: {value}</div>
		<br /> 
	-->

	<Keypad {keys} {mode} {bkspc_handler} {enter_handler} {press_handler} />
{:else}
	{#if show_pin}
		<div>Entered PIN: {value}</div>
	{/if}
	<br />
	<button on:click={() => (show_pin = !show_pin)}>Toggle PIN Visibility</button>
{/if}

<style>
	h1 {
		width: 100%;
	}
	h2 {
		width: 100%;
		display: flex;
		align-items: center;
		justify-content: center;
	}
	.colorbox {
		display: inline-block;
		height: 1rem;
		width: 1rem;
	}
</style>
