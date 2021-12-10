<script context="module">
	export const prerender = true;
</script>

<script>
	import GraphPaper from '$lib/GraphPaper.svelte';
</script>

<svelte:head>
	<title>Graph paper programming</title>
</svelte:head>

<section>
	<GraphPaper />
</section>

<style>
	section {
		display: flex;
		flex-direction: column;
		justify-content: center;
		align-items: center;
		flex: 1;
	}
</style>
