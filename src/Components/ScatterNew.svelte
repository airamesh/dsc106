<script>
  import { select, scaleLinear, extent, Delaunay, pointer, axisBottom, axisLeft } from 'd3';
  import { diabetesData } from "../output.js";

  // Setup dimensions and margins
  const margin = { top: 50, right: 30, bottom: 50, left: 60 };
  const width = 960 - margin.left - margin.right;
  const height = 500 - margin.top - margin.bottom;

  // Append SVG to the body
  const svg = select('body').append('svg')
      .attr('width', width + margin.left + margin.right)
      .attr('height', height + margin.top + margin.bottom)
    .append('g')
      .attr('transform', `translate(${margin.left}, ${margin.top})`);

  // Scales for the scatter plot
  const x = scaleLinear()
      .domain(extent(diabetesData, d => d.Glucose))
      .range([0, width]);

  const y = scaleLinear()
      .domain(extent(diabetesData, d => d.Insulin))
      .range([height, 0]);

  // Axes
  svg.append('g')
      .attr('transform', `translate(0, ${height})`)
      .call(axisBottom(x));

  svg.append('g')
      .call(axisLeft(y));

  // Tooltip div
  const tooltip = select('body').append('div')
      .attr('class', 'tooltip')
      .style('position', 'absolute')
      .style('text-align', 'center')
      .style('width', '200px')
      .style('height', 'auto')
      .style('padding', '10px')
      .style('background', 'white')
      .style('border', 'solid 1px #ccc')
      .style('border-radius', '5px')
      .style('pointer-events', 'none')
      .style('opacity', 0);

  // Points for scatter plot
  svg.selectAll('circle')
      .data(diabetesData)
      .enter().append('circle')
        .attr('cx', d => x(d.Glucose))
        .attr('cy', d => y(d.Insulin))
        .attr('r', 5)
        .style('fill', 'steelblue');

  // Voronoi for better mouse interaction
  const delaunay = Delaunay.from(diabetesData, d => x(d.Glucose), d => y(d.Insulin));
  const voronoi = delaunay.voronoi([0, 0, width, height]);

  svg.append('g')
    .selectAll('path')
    .data(diabetesData)
    .join('path')
      .attr('d', (_, i) => voronoi.renderCell(i))
      .style('fill', 'none')
      .style('pointer-events', 'all')
      .on('mouseover', (event, d) => {
        select(event.target).style('stroke', 'red');
        tooltip
          .style('opacity', 1)
          .style('left', `${event.pageX}px`)
          .style('top', `${event.pageY - 28}px`)
          .html(`Outcome: ${d.Outcome}<br>Glucose: ${d.Glucose}<br>Insulin: ${d.Insulin}`);
      })
      .on('mouseout', (event, d) => {
        select(event.target).style('stroke', 'none');
        tooltip.style('opacity', 0);
      });

</script>
