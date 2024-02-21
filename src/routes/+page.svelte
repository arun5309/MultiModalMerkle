<script lang="ts">
	import type { GetPointsResponse, CreateInstanceResponse, UpdateInstanceResponse } from './logic';
	import { make_random_puzzles, Mode, GameState, NCOLORS, get_ith_color } from './logic';
	import Keypad from './Keypad.svelte';

	let game_state = GameState.START;
	let keys = make_random_puzzles();
	let mode = Mode.DISPLAY;

	let value = '';
	let pin: string;
	$: pin = value;
	let round: number;
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

	function to_pos_num(val: string): number {
		if (val === '') return -1;
		return (Number(val) + 1) % 10;
	}

	let uid: string = '';
	let uid_valid: boolean = false;
	$: uid_valid = check_uid_valid(uid);
	let iid: number = -1;
	let actual_pin: string = '';

	function reset() {
		value = '';
		keys = make_random_puzzles();
		mode = Mode.DISPLAY;
		cur_pos = '';
		cur_color = '';

		game_state = GameState.START;
		show_pin = false;
		uid_valid = false;
		iid = -1;
		actual_pin = '';
		const temp = uid;
		uid = '';
		uid = temp;
	}

	async function get_points(): Promise<GetPointsResponse> {
		const url = `https://142.93.219.243.nip.io/points/${uid.toLowerCase()}`;
		const request = new Request(url, { method: 'GET' });
		const data = await fetch(request);
		return <GetPointsResponse>(<unknown>data.json());
	}

	function normalize() {
		uid = uid;
	}

	function set_actual_pin(val: string) {
		actual_pin = val;
	}

	function isAlphaNumeric(str: string): boolean {
		let code, i, len;
		for (i = 0, len = str.length; i < len; i++) {
			code = str.charCodeAt(i);
			if (!(code > 47 && code < 58) && !(code > 64 && code < 91) && !(code > 96 && code < 123)) {
				return false;
			}
		}
		return true;
	}

	function check_uid_valid(uid_cand: string): boolean {
		return uid_cand.length === 6 && isAlphaNumeric(uid_cand);
	}

	function progress_transition() {
		if (!uid_valid) return;
		const url = 'https://142.93.219.243.nip.io/create_instance';
		const data = {
			game_id: 'MMM',
			user_id: uid.toLowerCase()
		};
		const request = new Request(url, {
			method: 'POST',
			body: JSON.stringify(data),
			headers: new Headers({
				'Content-Type': 'application/json; charset=UTF-8'
			})
		});
		fetch(request).then((create_instance_value_temp) => {
			create_instance_value_temp.json().then((temp) => {
				const create_instance_value = <CreateInstanceResponse>(<unknown>temp);
				iid = create_instance_value.iid;
				game_state = GameState.PROGRESS;
			});
		});
	}

	function finish_transition() {
		const url = 'https://142.93.219.243.nip.io/update_instance';
		const data = {
			iid_value: iid,
			result_pin: value
		};
		const request = new Request(url, {
			method: 'POST',
			body: JSON.stringify(data),
			headers: new Headers({
				'Content-Type': 'application/json; charset=UTF-8'
			})
		});
		fetch(request).then((instance_response_value_temp) => {
			instance_response_value_temp.json().then((temp) => {
				const instance_response_value = <UpdateInstanceResponse>(<unknown>temp);
				if (iid !== instance_response_value.iid) {
					alert('Reached invalid state, please report bug!');
				}
				game_state = GameState.FINISH;
			});
		});
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
				if (value.length >= 4) {
					break;
				}
				if (!cur_color) {
					break;
				}
				value = value + String(keys[Number(cur_pos)][to_color_num(cur_color)].digit);
				if (value.length === 4) {
					finish_transition();
					break;
				}
				keys = make_random_puzzles();
				mode = Mode.DISPLAY;
				cur_pos = '';
				cur_color = '';
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
	<input
		type="text"
		placeholder="User ID"
		bind:value={uid}
		on:change={normalize}
		maxlength="6"
		name="userid"
		id="userid"
	/>
	{#if uid_valid}
		{#await get_points()}
			<p>Validating User ID...</p>
		{:then get_points_value}
			{#if get_points_value.uid !== null}
				{(set_actual_pin(get_points_value.actual_pin), '')}
				<p style="color: green">Points: {get_points_value.points}</p>
			{:else}
				<p style="color: red">Invalid User ID!</p>
			{/if}
		{:catch error}
			{(console.log(error), '')}
			<p style="color: purple">Network Error: Unable to check validity of User ID!</p>
		{/await}
	{/if}
	<br />
	<button on:click={progress_transition}>Start Game</button>
{:else if game_state === GameState.PROGRESS}
	{#if round < 5}
		<h2>Round: {round}</h2>
		<br />

		<div>
			<!-- Cur Pos: {to_pos_num(cur_pos)} -->
			Selected Color:
			<div style="background: {get_ith_color(to_color_num(cur_color))};" class="colorbox"></div>
		</div>
		<br />

		<!-- Useful for debugging! 
		<div>PIN Cur: {value}</div>
		<br /> 
	-->

		<Keypad bind:cur_pos {keys} {mode} {bkspc_handler} {enter_handler} {press_handler} />
	{/if}
{:else}
	{#if show_pin}
		<div>Entered PIN: {pin}</div>
	{/if}
	<br />
	{#if actual_pin === pin}
		<p style="color: green">Congratulations on entering the correct PIN!</p>
	{:else}
		<p style="color: red">Incorrect PIN entered, no points earned!</p>
	{/if}
	<br />
	{#await get_points()}
		<p>Fetching points...</p>
	{:then get_points_value}
		{#if get_points_value.uid !== null}
			<p style="color: green">Points: {get_points_value.points}</p>
		{:else}
			<p style="color: red">Invalid User ID!</p>
		{/if}
	{:catch error}
		{(console.log(error), '')}
		<p style="color: purple">Network Error: Unable to fetch points!</p>
	{/await}
	<br />
	<button on:click={() => (show_pin = !show_pin)}>Toggle Visibility of Entered PIN</button>
	<br />
	<button on:click={reset}>Play Again</button>
	<br />
	<button><a href="https://142.93.219.243.nip.io/">Checkout Other Games</a></button>
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
