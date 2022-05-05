<script>
	import * as d3 from "d3";
	import AxisHorizontal from "./AxisHorizontal.svelte";

	// access data
	let data = [];
	d3.csv(
	  "https://raw.githubusercontent.com/bryantkhoo/datavis-workshop-day-5/main/data/500_most_rated_books.csv"
	).then(res => {
	  data = res.slice(0, 500);
	});

	const xAccessor = d => parseFloat(d["average_rating"]);

	let xMetric = "Rating";

	// create chart dimensions
	let margin = {
	  left: 30,
	  top: 0,
	  right: 0,
	  bottom: 30
	};
	let width = 400;
	let height = 600;
	$: boundsWidth = width - margin.left - margin.right;
	$: boundsHeight = height - margin.top - margin.bottom;

	// create scales
	$: xScale = d3
	  .scaleLinear()
	  .domain(d3.extent(data, xAccessor))
	  .range([0, boundsWidth]);

	$: nodes = data.map(d => {
	  return {
	    data: d,
	    x: xScale(xAccessor(d)),
	    y: boundsHeight / 2,
	    r: 7
	  };
	});

	$: newNodes = [...nodes];

	$: d3.forceSimulation(nodes)
	  .force("collide", d3.forceCollide().radius(d => d.r + 1))
	  .on("tick", () => {
	    nodes = [...nodes];
	  });
</script>

<h2>500 of the most rated books on Goodreads</h2>

<figure>
  <div class="wrapper" bind:clientWidth={width} bind:clientHeight={height}>
    <svg width={width} height={height}>
      <g transform="translate({margin.left}, {margin.top})">
        {#each newNodes as d}
          <circle
            cx={d.x}
			cy={d.y}
            r="6"
            fill="#7897AB"
          />
        {/each}

      </g>
      <g transform="translate({margin.left}, {boundsHeight + margin.top })">
        <AxisHorizontal
          scale={xScale}
          count="5"
        />
      </g>
    </svg>
    <div
      class="label"
      style="transform: translate({boundsWidth}px, {boundsHeight + margin.top}px)">
      {xMetric}
    </div>
  </div>
</figure>

<style>
	.wrapper {
	  position: relative;
	  margin: 0;
	  font-family: sans-serif;
	  width: 100%;
	  height: 600px;
	  min-width: 0;
	}

	svg {
	  overflow: visible;
	}

	h2 {
	  text-align: center;
	  font-family: sans-serif;
	}

	.label {
	  position: absolute;
	  top: 0;
	  left: 0;
	  max-width: 4em;
	  font-size: 0.6em;
	}
</style>