<script>
    import { onMount } from 'svelte';
    import * as d3 from 'd3';
    
    export let scatterData = [];
    let chartRef;
    
    onMount(() => {
      const svg = d3.select(chartRef);
      const width = 800, height = 500;
      const margin = { top: 40, right: 30, bottom: 80, left: 60 };
    
      svg.attr("width", width).attr("height", height);
    
      const x = d3.scaleLinear()
        .domain([0, d3.max(scatterData, d => d.sales)])
        .range([margin.left, width - margin.right]);
    
      const y = d3.scaleLinear()
        .domain([0, d3.max(scatterData, d => d.profit)])
        .range([height - margin.bottom, margin.top]);
    
      // Axis labels and gridlines
      svg.append("g")
        .attr("transform", `translate(0,${height - margin.bottom})`)
        .call(d3.axisBottom(x));
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
        .text("Profit vs Sales");
    
      // X-axis label (Sales)
      svg.append("text")
        .attr("x", width / 2)
        .attr("y", height - margin.bottom + 50)
        .attr("text-anchor", "middle")
        .style("font-size", "14px")
        .style("font-weight", "bold")
        .text("Sales ($)");
    
      // Y-axis label (Profit)
      svg.append("text")
        .attr("transform", "rotate(-90)")
        .attr("x", -height / 2)
        .attr("y", margin.left - 40)
        .attr("text-anchor", "middle")
        .style("font-size", "14px")
        .style("font-weight", "bold")
        .text("Profit ($)");
    
      // Gridlines
      svg.append("g")
        .attr("class", "grid")
        .attr("transform", `translate(0,${height - margin.bottom})`)
        .call(d3.axisBottom(x)
          .ticks(5)
          .tickSize(-height + margin.top + margin.bottom)
          .tickFormat("")
        );
    
      svg.append("g")
        .attr("class", "grid")
        .attr("transform", `translate(${margin.left},0)`)
        .call(d3.axisLeft(y)
          .ticks(5)
          .tickSize(-width + margin.left + margin.right)
          .tickFormat("")
        );
    
      // Scatter plot points
      svg.selectAll("circle")
        .data(scatterData)
        .enter()
        .append("circle")
        .attr("cx", d => x(d.sales))
        .attr("cy", d => y(d.profit))
        .attr("r", 8)
        .attr("fill", "blue")
        .attr("stroke", "black")
        .attr("stroke-width", 1)
        .on("mouseover", (event, d) => {
          d3.select(event.target)
            .attr("fill", "orange");
          const tooltip = d3.select("#tooltip");
          tooltip.style("visibility", "visible")
            .text(`Sales: $${d.sales}, Profit: $${d.profit}`);
        })
        .on("mousemove", (event) => {
          const tooltip = d3.select("#tooltip");
          tooltip.style("top", `${event.pageY + 10}px`)
            .style("left", `${event.pageX + 10}px`);
        })
        .on("mouseout", (event) => {
          d3.select(event.target)
            .attr("fill", "blue");
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
  