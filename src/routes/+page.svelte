<script lang="ts">
	import FieldPlot from '$lib/FieldPlot.svelte';
</script>

<FieldPlot />

<style>
	html,
	body {
		margin: 0;
		padding: 0;
		width: 100%;
		height: 100%;
		overflow: hidden;
	}
	canvas {
		display: block;
		position: fixed;
		top: 0;
		left: 0;
		width: 100vw;
		height: 100vh;
	}
</style>
