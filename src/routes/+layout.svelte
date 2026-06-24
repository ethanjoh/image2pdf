<script lang="ts">
	import './layout.css';
	import favicon from '$lib/assets/favicon.svg';

	let { children } = $props();
</script>

<svelte:head>
	<title>Image2PDF</title>
	<link rel="icon" href={favicon} />
</svelte:head>
{@render children()}
