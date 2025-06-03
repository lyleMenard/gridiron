<script lang="ts">
	type Props = {
		name: string;
		min: number;
		max: number;
		value: number;
	};
	let { name, min, max, value = $bindable() }: Props = $props();
</script>

<div class="slidecontainer">
	<p>{name}: {value}</p>
	<input type="range" {min} {max} step={0.1} bind:value class="slider" id="myRange" />
</div>
