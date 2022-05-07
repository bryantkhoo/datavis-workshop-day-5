<script>
  import * as d3 from "d3";
  import AxisHorizontal from "./AxisHorizontal.svelte";
  import Tooltip from "./Tooltip.svelte";

  // access data
  let data = [];
  d3.csv(
    "https://raw.githubusercontent.com/bryantkhoo/datavis-workshop-day-5/main/data/500_most_rated_books.csv"
  ).then((res) => {
    data = res.slice(0, 100);
  });

  const xAccessor = (d) => parseFloat(d["average_rating"]);
  const sizeAccessor = (d) => parseInt(d["ratings_count"]);
  const authorAccessor = (d) => d["authors"];

  let xMetric = "Rating";

  // create chart dimensions
  let margin = {
    left: 30,
    top: 0,
    right: 0,
    bottom: 30,
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

  $: sizeScale = d3
    .scaleLinear()
    .domain(d3.extent(data, sizeAccessor))
    .range([5, 100]);

  $: authors = [...new Set(data.map((d) => d.authors))];

  $: colors = authors.map((_, i) => {
    console.log(d3.interpolateRainbow((i + 1) / authors.length));
    return d3.interpolateRainbow((i + 1) / authors.length);
  });

  $: colorScale = d3.scaleOrdinal().domain(authors).range(colors);
  $: console.log(d3.interpolateRainbow());
  $: console.log(colors);
  $: nodes = data.map((d) => {
    return {
      ...d,
      x: xScale(xAccessor(d)),
      y: boundsHeight / 2,
      r: sizeScale(sizeAccessor(d)),
    };
  });

  $: newNodes = [...nodes].map((node) => {
    node.x = Math.max(30, Math.min(690, node.x));
    node.y = Math.max(30, Math.min(600, node.y));
    return node;
  });

  $: d3.forceSimulation(nodes)
    .force(
      "collide",
      d3.forceCollide().radius((d) => d.r + 1)
    )
    .on("tick", () => {
      nodes = [...nodes];
    });

  let hoveredPoint = null;
  $: quadtree = d3
    .quadtree()
    .x((d) => d.x)
    .y((d) => d.y)
    .addAll(nodes);
</script>

<h1>100 of the most rated books on Goodreads</h1>

<figure>
  <div class="wrapper" bind:clientWidth={width} bind:clientHeight={height}>
    <svg {width} {height}>
      <g transform="translate({margin.left}, {margin.top})">
        {#each newNodes as d}
          <circle
            cx={d.x}
            cy={d.y}
            r={d.r}
            fill={hoveredPoint === d
              ? "skyblue"
              : colorScale(authorAccessor(d))}
          />
        {/each}
        <rect
          width={boundsWidth}
          height={boundsHeight}
          fill="transparent"
          on:mousemove={(e) => {
            const pos = d3.pointer(e);
            const x = pos[0];
            const y = pos[1];
            const closestPoint = quadtree.find(x, y);
            if (!closestPoint) return;
            const hoveredPointPosition = [closestPoint.x, closestPoint.y];
            // don't highlight if too far away
            // a^2 + b^2 = c^2
            const distance = Math.sqrt(
              (x - hoveredPointPosition[0]) ** 2 +
                (y - hoveredPointPosition[1]) ** 2
            );
            if (distance < 50) {
              hoveredPoint = closestPoint;
            } else {
              hoveredPoint = null;
            }
          }}
        />
      </g>
      <g transform="translate({margin.left}, {boundsHeight + margin.top})">
        <AxisHorizontal scale={xScale} count="5" />
      </g>
    </svg>
    <div
      class="label"
      style="transform: translate({boundsWidth}px, {boundsHeight +
        margin.top +
        15}px)"
    >
      {xMetric}
    </div>
    {#if hoveredPoint}
      <Tooltip
        x={margin.left + hoveredPoint.x}
        y={margin.top + hoveredPoint.y}
        placement={[
          hoveredPoint.x > boundsWidth - 50 ? -90 : -50,
          hoveredPoint.y < 80 ? 0 : -100,
        ]}
      >
        <strong>
          {hoveredPoint.title}
        </strong>
        <div>
          {hoveredPoint.authors}
        </div>
        <div>
          Rating: {hoveredPoint.average_rating}
        </div>
        <div>
          Ratings given: {hoveredPoint.ratings_count}
        </div>
      </Tooltip>
    {/if}
  </div>
</figure>

<style>
  .wrapper {
    position: relative;
    margin: 0;
    font-family: sans-serif;
    width: 100%;
    max-width: 750px;
    height: 600px;
    min-width: 0;
  }

  svg {
    overflow: visible;
  }

  h1 {
    text-align: center;
    font-family: "Raleway", sans-serif;
  }

  .label {
    position: absolute;
    top: 0;
    left: 0;
    max-width: 4em;
    font-size: 0.8em;
  }
</style>
