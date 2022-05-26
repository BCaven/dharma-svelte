<script>
    export let info = "this is the default";
    export let showInfo = true;
</script>

{info}
<button on:click = {() => showInfo = false}> return </button>

