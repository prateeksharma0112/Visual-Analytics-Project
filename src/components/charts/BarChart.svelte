<script>
    import * as d3 from 'd3';
    import { onMount } from 'svelte';

    export let data = [];

    let filteredData = [];
    let selectedRegion = "All Regions";
    let uniqueRegions = [];

    function filterData() {
        filteredData = selectedRegion === "All Regions"
            ? [...data]
            : data.filter(d => d.Region === selectedRegion);
        drawBarChart();  
    }

    onMount(() => {
        uniqueRegions = [...new Set(data.map(d => d.Region))];
        filteredData = [...data]; 
        drawBarChart(); 
    });

    function drawBarChart() {
        if (filteredData.length === 0) {
            d3.select("#barChart").html("<p>No data available for the selected region.</p>");
            return;
        }

        const salesByCategory = d3.rollups(
            filteredData,
            v => d3.sum(v, d => d.Sales),
            d => d.Category
        ).sort((a, b) => b[1] - a[1]); 

        const margin = { top: 50, right: 30, bottom: 60, left: 80 };
        const width = 600 - margin.left - margin.right;
        const height = 400 - margin.top - margin.bottom;

        // Clear previous chart
        const svg = d3.select("#barChart")
            .html('')
            .append("svg")
            .attr("width", width + margin.left + margin.right)
            .attr("height", height + margin.top + margin.bottom)
            .append("g")
            .attr("transform", `translate(${margin.left}, ${margin.top})`);

        // Define the Gradient for the Bars
        const gradient = svg.append("defs")
            .append("linearGradient")
            .attr("id", "barGradient")
            .attr("x1", "0%").attr("y1", "0%")
            .attr("x2", "0%").attr("y2", "100%");

        gradient.append("stop")
            .attr("offset", "0%")
            .attr("stop-color", "#4facfe");

        gradient.append("stop")
            .attr("offset", "100%")
            .attr("stop-color", "#00f2fe");

        // Define Scales
        const xScale = d3.scaleBand()
            .domain(salesByCategory.map(d => d[0]))
            .range([0, width])
            .padding(0.3);

        const yScale = d3.scaleLinear()
            .domain([0, d3.max(salesByCategory, d => d[1])])
            .range([height, 0]);

        // Draw Bars with the Gradient Color (No Hover Effect)
        svg.selectAll("rect")
            .data(salesByCategory)
            .enter()
            .append("rect")
            .attr("x", d => xScale(d[0]))
            .attr("y", height)
            .attr("width", xScale.bandwidth())
            .attr("height", 0)
            .style("fill", "url(#barGradient)")
            .attr("rx", 8)  // Rounded corners for a modern look
            .transition()
            .duration(800)
            .attr("y", d => yScale(d[1]))
            .attr("height", d => height - yScale(d[1]));

        // Value Labels on Top of Bars
        svg.selectAll(".label")
            .data(salesByCategory)
            .enter()
            .append("text")
            .attr("x", d => xScale(d[0]) + xScale.bandwidth() / 2)
            .attr("y", d => yScale(d[1]) - 10)
            .attr("text-anchor", "middle")
            .attr("font-size", "14px")
            .attr("font-weight", "bold")
            .attr("fill", "black")
            .text(d => `$${d[1].toFixed(2)}`);

        // X and Y Axes
        svg.append("g").attr("transform", `translate(0, ${height})`).call(d3.axisBottom(xScale));
        svg.append("g").call(d3.axisLeft(yScale));

        // Y-axis Label
        svg.append("text")
            .attr("transform", "rotate(-90)")
            .attr("y", -50)
            .attr("x", -height / 2)
            .attr("text-anchor", "middle")
            .style("font-size", "16px")
            .style("font-weight", "bold")
            .text("Total Sales (USD)");
    }
</script>

<!-- ✅ Stylish Region Filter Section -->
<section style="
    display: flex; 
    justify-content: center; 
    align-items: center; 
    gap: 1rem; 
    padding: 1rem;
    border-radius: 10px;
    background-color: #f3f4f6;
    box-shadow: 0px 4px 8px rgba(0,0,0,0.1);
">
    <label for="region" style="font-size: 1.1rem; font-weight: bold;">Select Region:</label>
    <select bind:value={selectedRegion} on:change={filterData} style="
        padding: 0.5rem;
        border: 2px solid #3498db;
        border-radius: 8px;
        cursor: pointer;
        font-size: 1rem;
        background: #f9f9f9;
    ">
        <option value="All Regions">All Regions</option>
        {#each uniqueRegions as region}
            <option value={region}>{region}</option>
        {/each}
    </select>
</section>

<!-- ✅ Bar Chart Container -->
<!-- <h3 style="text-align: center; margin-top: 2rem;">Sales by Category</h3> -->
<div id="barChart" style="padding: 1rem;"></div>
