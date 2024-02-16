<script lang="ts">
	export let style: string = "";
</script>

<button class="keyitem" {style} on:click><slot /></button>

<style>
	.keyitem {
		background: white;
		display: flex;
		align-items: center;
		justify-content: center;
        border: none;
	}
</style>
