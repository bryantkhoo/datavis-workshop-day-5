<script>
  export let scale;
  export let count;

  $: ticks = scale.ticks(count);
</script>

<line x1={scale.range()[1]} stroke="#444444" />
{#each ticks as x, i}
  <g transform="translate({scale(x)}, 0)">
    <text y="10" text-anchor="middle" dominant-baseline="hanging">
      {x}
    </text>
    <line y1="5" stroke="#444444" />
  </g>
{/each}

<style>
  text {
    font-size: 0.8em;
    fill: #444444;
    color: #444444;
  }
</style>
