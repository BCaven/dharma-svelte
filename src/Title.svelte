<script>
    export let mapId = 0;
    export let map = "Example map 1";
    export let start = false;
</script>

<h1> current id: {mapId} current map: {map} </h1>
{#if mapId < 2}
<button on:click={() => mapId++}> up </button>
{/if}
{#if mapId > 0}
<button on:click={() => mapId--}> down </button>
{/if}
<button on:click={() => start = true}> start </button>