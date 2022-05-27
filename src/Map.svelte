<script>
    import PotreeViewer from "./PotreeViewer.svelte";
    export let map = "default map";
    export let infoId;
    export let showInfo = false;
    export let start = false;
    function onInfoClick (id) {
        infoId = id;
        showInfo = true;
    }
</script>

{map}
<button on:click = {() => onInfoClick(0)}> Info 1 </button> <!-- to pass a function that has parameters to on:click you have to wrap it in another function-->
<button on:click = {() => onInfoClick(1)}> Info 2 </button>
<button on:click = {() => start = false}> return </button>

<PotreeViewer/>