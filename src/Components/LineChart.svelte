<script>
  import { line, curveStep } from "d3-shape";
  import { scaleLinear } from "d3-scale";
  import { diabetesData } from "../output.js";
  import { format } from "d3-format";

  const formatter = format(".0%");

  let height = 500;
  let width = 500;
  const mobile = window.innerWidth <= 700;
  const margin = {
    top: mobile ? 40 : 50,
    bottom: mobile ? 10 : 25,
    left: mobile ? 0 : 80,
    right: mobile ? 0 : 10,
  };

  $: xScale = scaleLinear()
    .domain([0, 14.4])
    .range([margin.left, width - margin.right]);
  $: accuracyScale = scaleLinear()
    .domain([0.0, 1])
    .range([height - margin.bottom, margin.top]);
  $: precisionScale = scaleLinear()
    .domain([0.0, 1])
    .range([height - margin.bottom, margin.top]);

  $: accuracyPath = line()
    .x((d) => xScale(d.thresh))
    .y((d) => accuracyScale(d.accuracy))
    .curve(curveStep);

  $: precisionPath = line()
    .x((d) => xScale(d.thresh))
    .y((d) => precisionScale(d.precision))
    .curve(curveStep);
</script>

<h1 class="body-header">Diabetes and Insulin</h1>
<p class="body-text">
  Insulin plays a crucial role in regulating blood sugar levels in the body. When we eat, our digestive system breaks down carbohydrates into glucose, which enters the bloodstream. In response, the pancreas releases insulin, a hormone that helps cells absorb glucose for energy or storage. <br>In individuals with diabetes, there are issues with insulin production or its effectiveness. In Type 1 diabetes, the pancreas produces little to no insulin due to autoimmune destruction of insulin-producing cells. In Type 2 diabetes, cells become resistant to insulin's effects, making it difficult for glucose to enter cells, and the pancreas may not produce enough insulin to compensate.
  The 2-hour serum insulin test measures insulin levels in the blood two hours after a glucose challenge. <br>In individuals with diabetes, this test can indicate whether insulin production is inadequate or if cells are resistant to insulin, helping healthcare providers diagnose and manage the condition effectively.
</p>

<div id="error-chart" bind:offsetWidth={width} bind:offsetHeight={height}>
  <svg
    width={width + margin.left + margin.right}
    height={height + margin.top + margin.bottom}
  >
    <!-- y-ticks -->
    {#each [0.2, 0.4, 0.6, 0.8, 1.0] as tick}
      <g transform={`translate(${margin.left - 5} ${accuracyScale(tick) + 0})`}>
        <!-- svelte-ignore component-name-lowercase -->
        <line
          class="y-axis-line"
          x1="0"
          x2={width - margin.right - margin.left}
          y1="0"
          y2="0"
          stroke="black"
        ></line>
        <text
          class="error-axis-text"
          y="0"
          text-anchor="end"
          dominant-baseline="middle">{formatter(tick)}</text
        >
      </g>
    {/each}
    <!-- axis lines -->
    <!-- x -->
    <!-- svelte-ignore component-name-lowercase -->
    <line
      class="error-axis-line"
      y1={height - margin.bottom}
      y2={height - margin.bottom}
      x1={margin.left}
      x2={width}
      stroke="black"
      stroke-width="2"
    ></line>
    <!-- y -->
    <!-- svelte-ignore component-name-lowercase -->
    <line
      class="error-axis-line"
      y1={margin.top}
      y2={height - margin.bottom}
      x1={margin.left}
      x2={margin.left}
      stroke="black"
      stroke-width="2"
    ></line>

    <path class="outline-line" d={accuracyPath(errorData)}></path>
    <path class="path-line" d={accuracyPath(errorData)} stroke="#c9208a"></path>
    <path class="outline-line" d={precisionPath(errorData)}></path>
    <path class="path-line" d={precisionPath(errorData)} stroke="#ab00d6"
    ></path>

    <!-- axis labels -->
    <text
      class="error-axis-label"
      y={height + margin.bottom}
      x={(width + margin.left) / 2}
      text-anchor="middle">Decision Boundary Threshold</text
    >
    <text
      class="error-axis-label"
      y={margin.left / 3}
      x={-(height / 2)}
      text-anchor="middle"
      transform="rotate(-90)">Score</text
    >

    <!-- x-ticks -->
    {#each xScale.ticks() as tick}
      <g transform={`translate(${xScale(tick) + 0} ${height - margin.bottom})`}>
        <text class="error-axis-text" y="15" text-anchor="end">{tick}</text>
      </g>
    {/each}
  </svg>
</div>

<style>
  #error-chart {
    margin: 2 auto;
    max-height: 55vh;
    width: 58%;
    margin: 3rem auto;
  }

  .error-axis-text {
    font-size: 0.9rem;
  }

  .y-axis-line {
    opacity: 0.2;
  }

  .error-axis-label {
    text-transform: uppercase;
    font-size: 1rem;
  }

  .path-line {
    fill: none;
    stroke-linejoin: round;
    stroke-linecap: round;
    stroke-width: 6;
  }

  .outline-line {
    fill: none;
    stroke: #f1f3f3;
    stroke-width: 10;
  }

  /* ipad */
  @media screen and (max-width: 950px) {
    #error-chart {
      max-height: 55vh;
      width: 85%;
      margin: 1rem auto;
    }
    .error-axis-label {
      font-size: 0.8rem;
    }
    .error-axis-text {
      font-size: 0.8rem;
    }
    .path-line {
      stroke-width: 5;
    }
    .outline-line {
      stroke-width: 9;
    }
  }
  /* mobile */
  @media screen and (max-width: 750px) {
    #error-chart {
      max-height: 55vh;
      width: 95%;
      margin: 1rem auto;
    }

    .error-axis-label {
      font-size: 0.75rem;
    }
    .error-axis-text {
      font-size: 0.7rem;
    }
    .path-line {
      stroke-width: 4;
    }
    .outline-line {
      stroke-width: 7;
    }
  }
</style>
