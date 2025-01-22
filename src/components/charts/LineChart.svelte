<script>
    import * as d3 from 'd3';
    import { onMount } from 'svelte';

    export let data = []; // Data passed to the component

    let filteredData = [];
    let selectedRegion = "All Regions";
    let uniqueRegions = [];

    // Filter Data Based on Region
    function filterData() {
        filteredData = selectedRegion === "All Regions"
            ? [...data]
            : data.filter(d => d.Region === selectedRegion);
        drawLineChart();
    }

    // On Mount: Prepare Unique Regions and Draw Initial Chart
    onMount(() => {
        uniqueRegions = [...new Set(data.map(d => d.Region))]; // Extract unique regions
        filteredData = [...data]; // Default: All data
        drawLineChart(); // Draw initial chart
    });

    // Draw Line Chart
    function drawLineChart() {
        if (filteredData.length === 0) {
            d3.select("#lineChart").html("<p>No data available for the selected region.</p>");
            return;
        }

        // Prepare Data for Line Chart
        const profitTrends = d3.rollups(
            filteredData,
            v => d3.sum(v, d => d.Profit),
            d => d3.timeFormat("%Y-%m")(new Date(d["Order Date"]))
        ).sort((a, b) => new Date(a[0]) - new Date(b[0]));

        const margin = { top: 50, right: 30, bottom: 60, left: 80 };
        const width = 600 - margin.left - margin.right;
        const height = 400 - margin.top - margin.bottom;

        // Clear previous chart
        const svg = d3.select("#lineChart")
            .html('')
            .append("svg")
            .attr("width", width + margin.left + margin.right)
            .attr("height", height + margin.top + margin.bottom)
            .append("g")
            .attr("transform", `translate(${margin.left}, ${margin.top})`);

        // Define Scales
        const xScale = d3.scaleTime()
            .domain(d3.extent(profitTrends, d => new Date(d[0]))) // Min and Max dates
            .range([0, width]);

        const yScale = d3.scaleLinear()
            .domain([0, d3.max(profitTrends, d => d[1])])
            .range([height, 0]);

        // Line Generator
        const line = d3.line()
            .x(d => xScale(new Date(d[0])))
            .y(d => yScale(d[1]))
            .curve(d3.curveCatmullRom); // Smooth curve

        // Gradient for Line
        const gradient = svg.append("defs")
            .append("linearGradient")
            .attr("id", "lineGradient")
            .attr("x1", "0%").attr("y1", "0%")
            .attr("x2", "100%").attr("y2", "0%");

        gradient.append("stop")
            .attr("offset", "0%")
            .attr("stop-color", "#4facfe");

        gradient.append("stop")
            .attr("offset", "100%")
            .attr("stop-color", "#00f2fe");

        // Draw Line
        svg.append("path")
            .datum(profitTrends)
            .attr("fill", "none")
            .attr("stroke", "url(#lineGradient)")
            .attr("stroke-width", 3)
            .attr("d", line);

        // Add Points
        svg.selectAll("circle")
            .data(profitTrends)
            .enter()
            .append("circle")
            .attr("cx", d => xScale(new Date(d[0])))
            .attr("cy", d => yScale(d[1]))
            .attr("r", 5)
            .attr("fill", "#4facfe");

        // Tooltips
        const tooltip = d3.select("#lineChart")
            .append("div")
            .style("position", "absolute")
            .style("background", "#fff")
            .style("border", "1px solid #ccc")
            .style("padding", "5px")
            .style("border-radius", "5px")
            .style("box-shadow", "0px 4px 8px rgba(0,0,0,0.1)")
            .style("display", "none");

        svg.selectAll("circle")
            .on("mouseover", (event, d) => {
                tooltip.style("display", "block")
                    .html(`<strong>Month:</strong> ${d[0]}<br><strong>Profit:</strong> $${d[1].toFixed(2)}`)
                    .style("left", `${event.pageX + 10}px`)
                    .style("top", `${event.pageY - 20}px`);
            })
            .on("mouseout", () => tooltip.style("display", "none"));

        // Add X-Axis with Yearly Labels
        svg.append("g")
            .attr("transform", `translate(0, ${height})`)
            .call(d3.axisBottom(xScale)
                .ticks(d3.timeYear.every(1)) // Show ticks for every year
                .tickFormat(d3.timeFormat("%Y"))); // Format as 'YYYY'

        // Optional: Styling for X-Axis labels
        svg.selectAll(".tick text")
            .style("font-size", "12px")
            .style("font-weight", "bold")
            .style("fill", "#333");

        // Add Y-Axis
        svg.append("g").call(d3.axisLeft(yScale));

        // Y-axis Label
        svg.append("text")
            .attr("transform", "rotate(-90)")
            .attr("y", -50)
            .attr("x", -height / 2)
            .attr("text-anchor", "middle")
            .style("font-size", "16px")
            .style("font-weight", "bold")
            .text("Profit (USD)");
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

<div id="lineChart" style="padding: 1rem;"></div>
