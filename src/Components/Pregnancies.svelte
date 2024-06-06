<script>
    import { scaleLinear, scaleBand } from 'd3-scale';
    import { max, bin } from 'd3-array';
    import { diabetesData } from "../output.js";
  
    let height = 800;  // SVG height
    let width = 1000;  // SVG width
    const margin = { top: 30, right: 30, bottom: 70, left: 80 };
  
    // Tooltip data
    let tooltip = { visible: false, text: '', x: '0px', y: '0px' };
  
    function showTooltip(event, data) {
      tooltip = {
        visible: true,
        text: `${data.type}: ${data.percentage.toFixed(2)}%`,
        x: `${event.pageX}px`,
        y: `${event.pageY - 28}px`
      };
    }
  
    function hideTooltip() {
      tooltip.visible = false;
    }
  
    // Filter data for diabetic and non-diabetic patients
    const diabeticData = diabetesData.filter(d => d.Outcome === '1');
    const healthyData = diabetesData.filter(d => d.Outcome === '0');
  
    const totalDiabetic = diabeticData.length;
    const totalHealthy = healthyData.length;
  
    // Create bins for the number of pregnancies
    const binGenerator = bin()
      .domain([0, max(diabetesData, d => +d.Pregnancies)])
      .thresholds(10);
  
    const diabeticBins = binGenerator(diabeticData.map(d => +d.Pregnancies));
    const healthyBins = binGenerator(healthyData.map(d => +d.Pregnancies));
  
    const xScale = scaleBand()
      .domain(diabeticBins.map(d => d.x0))
      .range([margin.left, width - margin.right])
      .padding(0.1);
  
    const yScale = scaleLinear()
      .domain([0, 1])
      .range([height - margin.bottom, margin.top]);
  
    const colorScale = d => d.Outcome === '1' ? 'red' : 'blue';
  </script>
  
  <h1 class="body-header">Normalized Distribution of Number of Pregnancies for Women With and Without Diabetes</h1>
  <p class="body-text">
    This histogram shows the normalized distribution of the number of pregnancies for diabetic patients versus healthy individuals.An increase in the number of pregnancies can elevate the risk of developing diabetes, particularly gestational diabetes. Each subsequent pregnancy adds strain to the body's ability to regulate blood sugar levels, increasing the likelihood of insulin resistance. Repeated pregnancies, especially if associated with significant weight gain, can further elevate the risk of developing type 2 diabetes later in life. Therefore, multiple pregnancies can cumulatively contribute to a higher diabetes risk.
  </p>
  
  <div class="svg-container">
    <svg width={width} height={height}>
      <!-- X-axis -->
      <g transform={`translate(0, ${height - margin.bottom})`}>
        {#each xScale.domain() as tick}
          <line y2="6" x1={xScale(tick) + xScale.bandwidth() / 2} x2={xScale(tick) + xScale.bandwidth() / 2} stroke="black"/>
          <text y="20" x={xScale(tick) + xScale.bandwidth() / 2} text-anchor="middle">{tick}</text>
        {/each}
        <text x={width / 2} y={50} text-anchor="middle" class="axis-label">
          Number of Pregnancies
        </text>
      </g>
  
      <!-- Y-axis -->
      <g transform={`translate(${margin.left}, 0)`}>
        {#each yScale.ticks(10) as tick}
          <line x2="-6" y1={yScale(tick)} y2={yScale(tick)} stroke="black"/>
          <text x="-10" y={yScale(tick) + 5} text-anchor="end">{tick * 100}%</text>
        {/each}
        <text x="-270" y="-60" text-anchor="middle" transform="rotate(-90)" class="axis-label">
          Percentage of Disease Status Group
        </text>
      </g>
  
      <!-- Diabetic bars -->
      {#each diabeticBins as bin}
        <rect
          x={xScale(bin.x0)}
          y={yScale(bin.length / totalDiabetic)}
          width={xScale.bandwidth() / 2}
          height={height - margin.bottom - yScale(bin.length / totalDiabetic)}
          fill="red"
          on:mouseenter={event => showTooltip(event, { type: 'Diabetic', percentage: (bin.length / totalDiabetic) * 100 })}
          on:mouseleave={hideTooltip}
        />
      {/each}
  
      <!-- Healthy bars -->
      {#each healthyBins as bin}
        <rect
          x={xScale(bin.x0) + xScale.bandwidth() / 2}
          y={yScale(bin.length / totalHealthy)}
          width={xScale.bandwidth() / 2}
          height={height - margin.bottom - yScale(bin.length / totalHealthy)}
          fill="blue"
          on:mouseenter={event => showTooltip(event, { type: 'Healthy', percentage: (bin.length / totalHealthy) * 100 })}
          on:mouseleave={hideTooltip}
        />
      {/each}
  
      <!-- Legend -->
      <g transform={`translate(${width - margin.right - 100}, ${margin.top})`}>
        <rect width="10" height="10" fill="red"></rect>
        <text x="15" y="10" text-anchor="start" alignment-baseline="middle">Diabetic</text>
        <rect y="20" width="10" height="10" fill="blue"></rect>
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
  