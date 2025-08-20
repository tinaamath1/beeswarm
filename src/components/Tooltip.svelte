<script>
    export let data;
    export let colorScale;
    export let width;

    let tooltipWidth;

    const xNudge = 5; 
    const yNudge = 5; 

    $: xPosition = 
        data.x + tooltipWidth + xNudge > width
        ? data.x - tooltipWidth - xNudge
        : data.x +xNudge; 
    $: yPosition = 
        data.y + yNudge;

    import { fly, fade } from "svelte/transition"
</script>

<div class='tooltip' 
in:fly={{y:20, duration: 200, delay:200}}
out:fade
style="
    top: {yPosition}px;
    left: {xPosition}px"
bind:clientWidth={tooltipWidth}
>
    <h1>
        {data.Country}
    </h1>
    <div class="info">
        <span class="score">
            {data["Are religiously affiliated"]}%
        </span>
        <span class="continent" style="background: {colorScale(data.Continent)}">
            {data.Continent}
        </span>
    </div>
    
    <!-- inline bar chart -->
     <span class="bar background"></span>
     <span class="bar foreground" 
        style="width:{data["Are religiously affiliated"]}%;
        background: {colorScale(data.Continent)}"></span>

</div>

<style>
    .tooltip{
        position: absolute;
        background: white; 
        box-shadow: rgba(0,0,0,.15) 2px 3px 8px;
        padding: 8px 6px; 
        border-radius: 3px; 
        pointer-events: none;
        transition: top 300ms ease, left 300ms ease
    }
    .info{
        display: flex; 
        justify-content: space-between;
        column-gap: 8px;
    }
    .score{
        font-size: .8rem; 
    }
    .continent{
        font-size: .65rem; 
        padding: 3px 4px 2px 4px; 
        border-radius: 3px; 
        text-transform: uppercase; 
        white-space: nowrap;
    }
    h1{
        margin: 0; 
        font-size: 1rem; 
        font-weight: 500;
        margin-bottom: 3px;
    }
    .bar{
        position: absolute; 
        bottom: 0;
        left: 0;
        height: 3px; 
        width: 100%;
    }
    .bar.background{
        background: #eee;
    }
</style>