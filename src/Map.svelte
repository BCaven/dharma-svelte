<script>
    export let map = "default map";
    export let infoId;
    export let showInfo = false;

    function onInfoClick (id) {
        infoId = id;
        showInfo = true;

    }
</script>

{map}
<button on:click = {onInfoClick(0)}> Info 1 </button>
<button on:click = {onInfoClick(1)}> Info 2 </button>