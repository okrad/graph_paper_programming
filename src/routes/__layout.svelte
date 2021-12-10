<script>
	import Header from '$lib/header/Header.svelte';
</script>

<Header />

<main>
	<slot />
</main>

<style>
	main {
		font-family: Arial, Helvetica, sans-serif;
		flex: 1;
		display: flex;
		flex-direction: column;
		padding: 1rem;
		width: 100%;
		max-width: 1360px;
		margin: 0 auto;
		box-sizing: border-box;
	}
</style>
