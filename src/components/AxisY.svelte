<script>
    export let yScale;
    export let groupByContinent

    import { fade } from "svelte/transition"
    
    $: ticks = yScale.domain();
</script>

<g class="axis y">
    {#each ticks as tick, index}
        {#if groupByContinent}
            <g class="tick" in:fade={{delay: index * 100 }} out:fade={{duration: 200}}>
                <text y={yScale(tick)}>{tick}</text>
            </g>
        {/if}
    {/each}
</g>