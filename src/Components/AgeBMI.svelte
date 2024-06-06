<script>
  import { scaleLinear } from 'd3-scale';
  import { max } from 'd3-array';
  import { Delaunay } from 'd3-delaunay';
  import { diabetesData } from "../output.js";

  let tooltip = { visible: false, text: '', x: '0px', y: '0px' };

  function showTooltip(event, data) {
    tooltip = {
      visible: true,
      text: `Outcome: ${data.Outcome === '1' ? 'Diabetic' : 'Non-Diabetic'}, Age: ${data.Age}, BMI: ${data.BMI}`,
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
  let ageScale = scaleLinear()
    .domain([20, max(diabetesData, d => +d.Age)])  // Start at age 20
    .range([margin.left, width - margin.right]);

  let bmiScale = scaleLinear()
    .domain([0, max(diabetesData, d => +d.BMI)])
    .range([height - margin.bottom, margin.top]);

  let delaunay = Delaunay.from(diabetesData, 
    d => ageScale(d.Age), 
    d => bmiScale(d.BMI)
  );
  let voronoi = delaunay.voronoi([0, 0, width, height]);

  let colorScale = d => +d.Outcome === 1 ? 'orange' : 'green';
</script>

<h1 class="body-header">Age vs BMI for those with and without Diabetes</h1>
<p class="body-text">
  Diabetes, Age, and BMI (Body Mass Index) are interrelated factors that influence each other significantly. As age increases, the risk of developing type 2 diabetes rises due to the body's declining ability to regulate blood sugar levels and increasing insulin resistance. BMI, a measure of body fat based on height and weight, is also a critical factor: higher BMI values indicate overweight or obesity, which are major risk factors for diabetes. Excess body fat, especially around the abdomen, can lead to insulin resistance, further increasing the likelihood of developing diabetes. Thus, older age and higher BMI both contribute to a greater risk of diabetes, often compounding each other's effects.
</p>

<div class="svg-container">
  <svg width={width} height={height}>
    <!-- Manual x-axis -->
    <g transform={`translate(0, ${height - margin.bottom})`}>
      {#each ageScale.ticks(10) as tick}
        <line y2="6" x1={ageScale(tick)} x2={ageScale(tick)} stroke="black"/>
        <text y="20" x={ageScale(tick)} text-anchor="middle">{tick}</text>
      {/each}
      <text x={width / 2} y={50} text-anchor="middle" class="axis-label">
        Age (years)
      </text>
    </g>

    <!-- Manual y-axis -->
    <g transform={`translate(${margin.left}, 10)`}>
      {#each bmiScale.ticks(10) as tick}
        <line x2="-6" y1={bmiScale(tick)} y2={bmiScale(tick)} stroke="black"/>
        <text x="-10" y={bmiScale(tick) + 5} text-anchor="end">{tick}</text>
      {/each}
    </g>
    <text x={-350} y={0} text-anchor="middle" transform="rotate(-90)" class="axis-label">
      Body Mass Index (kg / m^2)
    </text>

    <!-- Scatterplot points -->
    {#each diabetesData as data}
      <circle
        cx={ageScale(data.Age)}
        cy={bmiScale(data.BMI)}
        r="5"
        fill={colorScale(data)}
        on:mouseenter={event => showTooltip(event, data)}
        on:mouseleave={hideTooltip}
      />
    {/each}

    <!-- Voronoi overlay -->
    <g class="voronoi">
      {#each diabetesData as data, i}
        <path 
          d={voronoi.renderCell(i)} 
          fill="none" 
          pointer-events="all"
          on:mousemove={event => showTooltip(event, data)}
          on:mouseleave={hideTooltip}
        />
      {/each}
    </g>

    <!-- Legend -->
    <g transform={`translate(${width - margin.right - 100}, ${margin.top})`}>
      <rect width="10" height="10" fill="orange"></rect>
      <text x="15" y="10" text-anchor="start" alignment-baseline="middle">Diabetic</text>
      <rect y="20" width="10" height="10" fill="green"></rect>
      <text x="15" y="30" text-anchor="start" alignment-baseline="middle">Healthy</text>
    </g>
  </svg>
  {#if tooltip.visible}
    <div class="tooltip" style="left: {tooltip.x}; top: {tooltip.y};">
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
