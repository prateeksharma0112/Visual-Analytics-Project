<script>
  import { onMount } from 'svelte';
  import * as d3 from 'd3';
  
  export let lineData = [];
  let chartRef;
  
  onMount(() => {
    const svg = d3.select(chartRef);
    const width = 800, height = 500;
    const margin = { top: 40, right: 30, bottom: 80, left: 60 };
  
    svg.attr("width", width).attr("height", height);
  
    const x = d3.scaleTime()
      .domain(d3.extent(lineData, d => d.date))
      .range([margin.left, width - margin.right]);
  
    const y = d3.scaleLinear()
      .domain([0, d3.max(lineData, d => d.sales)])
      .range([height - margin.bottom, margin.top]);
  
    const line = d3.line()
      .x(d => x(d.date))
      .y(d => y(d.sales));
  
    // Axis labels and gridlines
    svg.append("g")
      .attr("transform", `translate(0,${height - margin.bottom})`)
      .call(d3.axisBottom(x).ticks(d3.timeMonth.every(1)).tickFormat(d3.timeFormat("%b"))); // Format months
    svg.append("g")
      .attr("transform", `translate(${margin.left},0)`)
      .call(d3.axisLeft(y));
  
    // Title for the chart
    svg.append("text")
      .attr("x", width / 2)
      .attr("y", margin.top - 10)
      .attr("text-anchor", "middle")
      .style("font-size", "18px")
      .style("font-weight", "bold")
      .text("Sales Trend Over Time (2024)");
  
    // X-axis label (Date)
    svg.append("text")
      .attr("x", width / 2)
      .attr("y", height - margin.bottom + 50)
      .attr("text-anchor", "middle")
      .style("font-size", "14px")
      .style("font-weight", "bold")
      .text("Month");
  
    // Y-axis label (Sales)
    svg.append("text")
      .attr("transform", "rotate(-90)")
      .attr("x", -height / 2)
      .attr("y", margin.left - 40)
      .attr("text-anchor", "middle")
      .style("font-size", "14px")
      .style("font-weight", "bold")
      .text("Sales ($)");
  
    // Line path
    svg.append("path")
      .data([lineData])
      .attr("class", "line")
      .attr("d", line)
      .attr("fill", "none")
      .attr("stroke", "green")
      .attr("stroke-width", 2);
  
    // Data points (circles)
    svg.selectAll("circle")
      .data(lineData)
      .enter()
      .append("circle")
      .attr("cx", d => x(d.date))
      .attr("cy", d => y(d.sales))
      .attr("r", 5)
      .attr("fill", "green")
      .on("mouseover", (event, d) => {
        d3.select(event.target)
          .attr("fill", "yellow");
        const tooltip = d3.select("#tooltip");
        tooltip.style("visibility", "visible")
          .text(`Date: ${d3.timeFormat("%b %d, %Y")(d.date)}, Sales: $${d.sales}`);
      })
      .on("mousemove", (event) => {
        const tooltip = d3.select("#tooltip");
        tooltip.style("top", `${event.pageY + 10}px`)
          .style("left", `${event.pageX + 10}px`);
      })
      .on("mouseout", (event) => {
        d3.select(event.target)
          .attr("fill", "green");
        d3.select("#tooltip").style("visibility", "hidden");
      });
  
    // Tooltips
    const tooltip = d3.select("body").append("div")
      .attr("id", "tooltip")
      .style("position", "absolute")
      .style("background-color", "rgba(0,0,0,0.6)")
      .style("color", "white")
      .style("padding", "5px")
      .style("border-radius", "5px")
      .style("visibility", "hidden");
  });
</script>
  
<svg bind:this={chartRef}></svg>
