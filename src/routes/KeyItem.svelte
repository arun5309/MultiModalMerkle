<script lang="ts">
	export let style: string = "";
</script>

<div class="keyitem" {style}><slot /></div>

<style>
	.keyitem {
		background: white;
		display: flex;
		align-items: center;
		justify-content: center;
	}
</style>
