<script>
  import { scaleLinear } from 'd3-scale';
  import { max} from 'd3-array';
  import { Delaunay } from 'd3-delaunay';
  import { diabetesData } from "../output.js";

  let tooltip = { visible: false, text: '', x: '0px', y: '0px' };

  // function showTooltip(event, data) {
  //   tooltip = {
  //     visible: true,
  //     text: `${data.Outcome === '1' ? 'Diabetic' : 'Non-Diabetic'}\nInsulin: ${data.Insulin}\nGlucose: ${data.Glucose}`,
  //     x: event.clientX , // Offset tooltip position to avoid cursor overlap
  //     y: event.clientY
  //   };
  // }

  function showTooltip(event, data) {
    tooltip = {
      visible: true,
      text: `Outcome: ${data.Outcome === '1' ? 'Diabetic' : 'Non-Diabetic'},  Insulin: ${data.Insulin} Glucose: ${data.Glucose}`,
      x: `${event.pageX}px`,
      y: `${event.pageY - 28}px`
    };
  }

  function hideTooltip() {
    tooltip.visible = false;
  }

  let height = 800;  // SVG height
  let width = 1000;   // SVG width
  const margin = { top: 30, right: 30, bottom: 70, left: 80 }; 

  // Scales
  let glucoseScale = scaleLinear()
    .domain([0, max(diabetesData, d => +d.Glucose)])
    .range([margin.left, width - margin.right]);

  let insulinScale = scaleLinear()
    .domain([0, max(diabetesData, d => +d.Insulin)])
    .range([height - margin.bottom, margin.top]);

  let delaunay = Delaunay.from(diabetesData, 
    d => glucoseScale(d.Glucose), 
    d => insulinScale(d.Insulin)
  );
  let voronoi = delaunay.voronoi([0, 0, width, height]);

  let colorScale = d => +d.Outcome === 1 ? 'red' : 'blue';
</script>

<h1 class="body-header">Diabetes and Insulin</h1>
<p class="body-text">
  Insulin plays a crucial role in regulating blood sugar levels in the body. When we eat, our digestive system breaks down carbohydrates into glucose, which enters the bloodstream. In response, the pancreas releases insulin, a hormone that helps cells absorb glucose for energy or storage. <br>In individuals with diabetes, there are issues with insulin production or its effectiveness. In Type 1 diabetes, the pancreas produces little to no insulin due to autoimmune destruction of insulin-producing cells. In Type 2 diabetes, cells become resistant to insulin's effects, making it difficult for glucose to enter cells, and the pancreas may not produce enough insulin to compensate.
  The 2-hour serum insulin test measures insulin levels in the blood two hours after a glucose challenge. <br>In individuals with diabetes, this test can indicate whether insulin production is inadequate or if cells are resistant to insulin, helping healthcare providers diagnose and manage the condition effectively.
</p>

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
  padding: 5px;
  background: rgb(244, 226, 226);
  border: 1px solid rgb(0, 0, 0);
  pointer-events: none;
  z-index: 100;
  white-space: pre-line;
  transform: translateX(-50%); 
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
