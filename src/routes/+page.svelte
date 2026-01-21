<script lang="ts">
	import MazeGame from '$lib/components/MazeGame.svelte';
</script>

<svelte:head>
	<title>Maze Runner - Progressive Maze Game</title>
	<meta
		name="description"
		content="Navigate through 25 challenging mazes! Progressive difficulty, star ratings, touch controls for mobile. Test your skills from easy 5x5 to massive 25x20 mazes."
	/>
	<meta
		name="viewport"
		content="width=device-width, initial-scale=1, maximum-scale=1, user-scalable=no"
	/>
	<meta name="theme-color" content="#0f172a" />
	<meta name="apple-mobile-web-app-capable" content="yes" />
	<meta name="apple-mobile-web-app-status-bar-style" content="black-translucent" />

	<!-- Open Graph -->
	<meta property="og:title" content="Maze Runner - Progressive Maze Game" />
	<meta
		property="og:description"
		content="Navigate through 25 challenging mazes! Progressive difficulty, star ratings, and touch controls."
	/>
	<meta property="og:type" content="website" />
	<meta property="og:url" content="https://mazes.nickwinter.net" />

	<!-- Twitter Card -->
	<meta name="twitter:card" content="summary_large_image" />
	<meta name="twitter:title" content="Maze Runner - Progressive Maze Game" />
	<meta
		name="twitter:description"
		content="Navigate through 25 challenging mazes! Progressive difficulty, star ratings, and touch controls."
	/>
</svelte:head>

<main class="min-h-screen bg-slate-900">
	<MazeGame />
</main>
