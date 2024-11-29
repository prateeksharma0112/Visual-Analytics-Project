<script>
    import { onMount } from 'svelte';
    import * as d3 from 'd3';
  
    export let barData = [];
    let chartRef;
  
    onMount(() => {
      const svg = d3.select(chartRef);
      const width = 600, height = 400;
      const margin = { top: 40, right: 30, bottom: 60, left: 60 };
  
      svg.attr("width", width).attr("height", height);
  
      const x = d3.scaleBand()
        .domain(barData.map(d => d.category))
        .range([margin.left, width - margin.right])
        .padding(0.1);
  
      const y = d3.scaleLinear()
        .domain([0, d3.max(barData, d => d.sales)])
        .nice()
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
        .text("Total Sales by Category");
  
      // X-axis label (Category)
      svg.append("text")
        .attr("x", width / 2)
        .attr("y", height - margin.bottom + 40)
        .attr("text-anchor", "middle")
        .style("font-size", "14px")
        .style("font-weight", "bold")
        .text("Category");
  
      // Y-axis label (Sales)
      svg.append("text")
        .attr("transform", "rotate(-90)")
        .attr("x", -height / 2)
        .attr("y", margin.left - 40)
        .attr("text-anchor", "middle")
        .style("font-size", "14px")
        .style("font-weight", "bold")
        .text("Sales ($)");
  
      // Bars
      svg.selectAll("rect")
        .data(barData)
        .enter()
        .append("rect")
        .attr("x", d => x(d.category))
        .attr("y", d => y(d.sales))
        .attr("width", x.bandwidth())
        .attr("height", d => y(0) - y(d.sales))
        .attr("fill", "steelblue");
    });
  </script>
  
  <svg bind:this={chartRef}></svg>  