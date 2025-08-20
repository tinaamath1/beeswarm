<script>
    export let colorScale; 
    export let hoveredContinent;
    // console.log(colorScale.range());
    // $: console.log({hoveredContinent})
</script>

<div class="legend" on:mouseleave={()=> (hoveredContinent = null)}>
    {#each colorScale.domain() as continent}
    <!-- svelte-ignore a11y-mouse-events-have-key-events -->
    <p on:mouseover={() => hoveredContinent = continent}
        class:unhovered = {hoveredContinent && hoveredContinent !== continent}>
        <span style="background-color: {colorScale(continent)}"></span>
        {continent}
    </p>
    {/each}

</div>

<style>
    .legend{
        display: flex; 
        flex-direction: row; 
        justify-content: center;
        flex-wrap: wrap;
        column-gap: 10px;
        row-gap: 5px; 
        margin-bottom: 10px; 
    }

    span{
        width: 9px; 
        height: 9px; 
        display: inline-block;
        border-radius: 50%;
        border: 1px solid rgba(0,0,0,0.5)
    }

    .unhovered{
        opacity: 0.3
    }

    p{
        font-size: 14px; 
        display: flex; 
        align-items: center; 
        column-gap: 3px;
        transition: opacity 300ms ease;
        cursor: pointer;
    }
</style>