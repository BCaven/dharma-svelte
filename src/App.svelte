<script>
	//import PotreeViewer from "./PotreeViewer.svelte";
	import Title from "./Title.svelte";
	import Map from "./Map.svelte";
	import Info from "./Info.svelte";
	let mapId = 0; // need to add boundaries for these so they do not go over/under the expected ids
	let infoId = 0;
	let maps = [ // these are place-holders, will need to change them to jsons for the actual thing
		"example map 1",
		"example map 2",
		"example map 3"
	];
	let infos = [
		"example info 1",
		"example info 2"
	];
	let start = false;
	let showInfo = false;
	$: map = maps[mapId];
	$: info = infos[infoId];
	
	
</script>


<main>
	<h1> But did it build? </h1>
	
	{#if !start}
		<Title {map} bind:mapId bind:start/>
	{:else}
		{#if !showInfo}
			<Map map = {map} bind:infoId bind:showInfo bind:start/>
		{/if}
	{/if}

	{#if showInfo} <!-- could totally put this in the map file... but it feels better to have it all controlled by the app-->
		<Info {info} bind:showInfo/>
	{/if}
</main>

<style>
	main {
		text-align: center;
		padding: 1em;
		max-width: 240px;
		margin: 0 auto;
	}

	h1 {
		color: #ff3e00;
		text-transform: uppercase;
		font-size: 4em;
		font-weight: 100;
	}

	@media (min-width: 640px) {
		main {
			max-width: none;
		}
	}
</style>