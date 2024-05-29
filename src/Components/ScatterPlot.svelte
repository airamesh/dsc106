<script>
  import { scaleLinear } from 'd3-scale';
  import { max } from 'd3-array';
  import { diabetesData } from "../output.js";

  let tooltip = { visible: false, text: '', x: 0, y: 0 };

  function showTooltip(event, data) {
    tooltip = {
      visible: true,
      text: `${data.Outcome === '1' ? 'Diabetic' : 'Non-Diabetic'}\nInsulin: ${data.Insulin}\nGlucose: ${data.Glucose}`,
      x: event.clientX , // Offset tooltip position to avoid cursor overlap
      y: event.clientY
    };
  }

  function hideTooltip() {
    tooltip.visible = false;
  }

  let height = 800;  // SVG height
  let width = 1000;   // SVG width
  const margin = { top: 30, right: 30, bottom: 70, left: 80 }; 

  // Scales
  $: glucoseScale = scaleLinear()
    .domain([0, max(diabetesData, d => +d.Glucose)])
    .range([margin.left, width - margin.right]);

  $: insulinScale = scaleLinear()
    .domain([0, max(diabetesData, d => +d.Insulin)])
    .range([height - margin.bottom, margin.top]);

  $: colorScale = d => +d.Outcome === 1 ? 'red' : 'blue';
</script>

<div class="svg-container">
  <svg width={width} height={height}>
    <!-- Manual x-axis -->
    <g transform={`translate(0, ${height - margin.bottom})`}>
      {#each glucoseScale.ticks(10) as tick}
        <line y2="6" x1={glucoseScale(tick)} x2={glucoseScale(tick)} stroke="black"/>
        <text y="20" x={glucoseScale(tick)} text-anchor="middle">{tick}</text>
      {/each}
      <text x={width / 2} y={50} text-anchor="middle" class="axis-label">
        Plasma Glucose Concentration (2 hrs post OGTT)
      </text>
    </g>

    <!-- Manual y-axis -->
    <g transform={`translate(${margin.left}, 10)`}>
      {#each insulinScale.ticks(10) as tick}
        <line x2="-6" y1={insulinScale(tick)} y2={insulinScale(tick)} stroke="black"/>
        <text x="-10" y={insulinScale(tick) + 5} text-anchor="end">{tick}</text>
      {/each}

      <!-- Adjusted position for better visibility -->
    </g>
    <text x={-350} y={0} text-anchor="middle" transform="rotate(-90)" class="axis-label">
      Insulin (2 hr serum, mu U/ml)
    </text>

    <!-- Scatterplot points -->
    {#each diabetesData as data}
      <!-- svelte-ignore a11y-no-static-element-interactions -->
      <circle
        cx={glucoseScale(data.Glucose)}
        cy={insulinScale(data.Insulin)}
        r="5"
        fill={colorScale(data)}
        on:mouseenter={event => showTooltip(event, data)}
        on:mouseleave={hideTooltip}
      />
    {/each}
  </svg>
  {#if tooltip.visible}
    <div class="tooltip" style="left: {tooltip.x}px; top: {tooltip.y}px;">
      {tooltip.text}
    </div>
  {/if}
</div>

<style>

.tooltip {
    position: absolute;
    padding: 10px;
    background: white;
    border: 1px solid #ccc;
    pointer-events: none;
    z-index: 10;
    white-space: pre-line;
  }
  .svg-container {
    display: flex;
    justify-content: center;
    margin-top: 20px; 
    overflow: visible;
  }
  svg {
    background-color: #f4f4f4;
    overflow: visible;
  }
  text, .axis-label {
    font-size: 12px;
    fill: black; 
  }
  .axis-label {
    font-weight: bold;
    font-size: 14px;
  }
  line {
    stroke: #bbb;
  }
  
</style>
