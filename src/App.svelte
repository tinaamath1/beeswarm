<script>
  import data from "$data/spiritual_beliefs.json";
  import { forceSimulation, forceX, forceY, forceCollide } from "d3-force";
  import { scaleLinear } from "d3-scale"; 
  import { scaleBand } from "d3-scale"
  import { fade } from "svelte/transition"
  console.log(data)

  let width = 400, 
    height = 600; 
  
  const margin = {top: 20, right: 30, bottom: 40, left: 0}

  $: innerWidth = width - margin.left - margin.right; 
  let innerHeight = height - margin.top - margin.bottom; 

  const RADIUS = 5; 

  import { mean, rollups } from "d3-array"

  const continents = rollups(
    data,
    v => mean(v, d => d["Are religiously affiliated"]),
    d => d["Continent"]
  ) 
    .sort((a, b) => a[1] - b[1]) 
    .map(d => d[0]); 
  
  let simulation = forceSimulation(data);


  let nodes = []; 
  simulation.on("tick", () => {
    nodes = simulation.nodes()
  })

  $: {
    simulation 
      .force("x", forceX().x(d => xScale(d["Are religiously affiliated"])).strength(0.8))
      .force("y", forceY()
        .y(d => 
          groupByContinent
            ? yScale(d.Continent)
            : innerHeight/2
        )
        .strength(0.2)
      )
      .force("collide", forceCollide().radius(d => radiusScale(d["Are religiously affiliated"])))
      .alpha(0.3)
      .alphaDecay(.0005)
      .restart()
  }

  $: xScale = scaleLinear() 
    // .domain(extent(data, d => d["Are religiously affiliated"]))
    .domain([0,100])
    .range([0, innerWidth])

    let yScale = scaleBand()
      .domain(continents)
      .range([innerHeight, 0])

  import { scaleOrdinal } from "d3-scale"; 

  const colorRange = ["#dda0dd", "#fe7f2d", "#fcca46", "#a1c181", "#619b8a", "#eae2b7"]
  let colorScale = scaleOrdinal()
    .domain(continents)
    .range(colorRange)

  import { scaleSqrt } from "d3-scale"
  let radiusScale = scaleSqrt() 
    .domain([0,100])
    .range(width < 568 ? [2, 6] : [3, 8])

  import AxisX from "$components/AxisX.svelte"
  import AxisY from "$components/AxisY.svelte"
  import Legend from "$components/Legend.svelte"

  let hovered; 
  // $: console.log(hovered);
  import Tooltip from "$components/Tooltip.svelte"

  let hoveredContinent;
  // $: console.log({hoveredContinent})

  let groupByContinent = false; 

</script>


<Legend {colorScale} bind:hoveredContinent/>
<div 
  class= 'chart-container' 
  bind:clientWidth={width}
  on:click={(e) => {
    groupByContinent = !groupByContinent; 
    hovered = null;
  }}>
  <svg {width} {height} on:mouseleave={()=> {hovered = null}}>
    <g class="inner-chart" transform="translate({margin.left},{margin.top})">
      <AxisX {xScale} height={innerHeight} width={innerWidth} />
      <AxisY {yScale} {groupByContinent}/>
      {#if hovered}
        <!-- svelte-ignore missing-declaration -->
        <line
          transition:fade
          x1 = {hovered.x}
          x2 = {hovered.x}
          y1 = {hovered.y}
          y2 = {height - margin.bottom}
          stroke = {colorScale(hovered.Continent)}
          stroke-width = "2"
        />
      {/if}
      {#each nodes as node}
      <!-- svelte-ignore a11y-no-noninteractive-tabindex -->
      <circle
        in:fade ={{delay:200}}
        cx= {node.x}
        cy= {node.y}
        r= {radiusScale(node["Are religiously affiliated"])}
        fill={colorScale(node.Continent)}
        title = {node.Country}
        stroke = {hovered || hoveredContinent 
                  ? hovered === node || hoveredContinent === node.Continent
                    ? "black"
                    : "transparent"
                  : "#00000090"}
        opacity = {hovered || hoveredContinent
                  ? hovered === node || hoveredContinent === node.Continent
                    ? 1 
                    : 0.3
                  : 1}
        on:mouseover={()=> {
          hovered = node;
        }}
        on:focus={()=> {
          hovered = node;
        }}
        tabindex="0"
        on:click={(e) => {
          e.stopImmediatePropagation();
        }}
        
      />
      {/each}
    </g>
  </svg>
  {#if hovered}
    <Tooltip
      data={hovered}
      {colorScale}
      {width}
    />
  {/if}
</div>

<style>

  :global(.tick text, .axis-title){
    font-size: 14px; 
    font-weight: 400; 
    fill: hsla(212, 10%, 53%, 1);
    user-select: none;
  }

  circle{
    transition: stroke 300ms ease, opacity 300ms ease; 
    cursor: pointer;
  }
</style>
