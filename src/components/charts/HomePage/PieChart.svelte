<script>
    import * as d3 from 'd3';
    import { onMount } from 'svelte';

    export let data = [];

    let filteredData = [];
    let selectedSegment = "All Segments";
    let uniqueSegments = [];

    function filterData() {
        filteredData = selectedSegment === "All Segments"
            ? [...data]
            : data.filter(d => d.Segment === selectedSegment);
        drawPieChart();
    }

    onMount(() => {
        uniqueSegments = [...new Set(data.map(d => d.Segment))];
        filteredData = [...data];
        drawPieChart();
    });

    function drawPieChart() {
        if (filteredData.length === 0) {
            d3.select("#pieChart").html("<p>No data available for the selected segment.</p>");
            return;
        }

        const profitByRegion = d3.rollups(
            filteredData,
            v => d3.sum(v, d => d.Profit),
            d => d.Region
        ).map(([key, value]) => ({ Region: key, Profit: value }));

        const totalProfit = d3.sum(profitByRegion, d => d.Profit);

        const margin = 40;
        const width = 400;
        const height = 400;
        const radius = Math.min(width, height) / 2 - margin;

        // Clear previous chart
        const svg = d3.select("#pieChart")
            .html("")
            .append("svg")
            .attr("width", width)
            .attr("height", height)
            .append("g")
            .attr("transform", `translate(${width / 2}, ${height / 2})`);

        const color = d3.scaleOrdinal()
            .domain(profitByRegion.map(d => d.Region))
            .range(["#ff9999", "#66b3ff", "#99ff99", "#ffcc99", "#c2c2f0", "#ffb3e6"]);

        const pie = d3.pie().value(d => d.Profit);
        const data_ready = pie(profitByRegion);

        const arcGenerator = d3.arc().innerRadius(0).outerRadius(radius);
        const arcHover = d3.arc().innerRadius(0).outerRadius(radius + 10);

        const tooltip = d3.select("#pieChart")
            .append("div")
            .style("position", "absolute")
            .style("background", "#fff")
            .style("border", "1px solid #ccc")
            .style("padding", "8px")
            .style("border-radius", "5px")
            .style("box-shadow", "0px 4px 8px rgba(0,0,0,0.1)")
            .style("display", "none");

        svg.selectAll("path")
            .data(data_ready)
            .enter()
            .append("path")
            .attr("d", arcGenerator)
            .attr("fill", d => color(d.data.Region))
            .style("opacity", 0.9)
            .on("mouseover", (event, d) => {
                // Highlight the slice
                d3.select(event.currentTarget)
                    .transition()
                    .duration(200)
                    .attr("d", arcHover);

                // Show tooltip near the slice corner
                const [x, y] = arcHover.centroid(d); // Get slice corner position
                tooltip.style("display", "block")
                    .style("left", `${event.pageX}px`)
                    .style("top", `${event.pageY - 40}px`)
                    .html(`
                        <strong>Region:</strong> ${d.data.Region}<br>
                        <strong>Profit:</strong> $${d.data.Profit.toFixed(2)}<br>
                        <strong>Contribution:</strong> ${(d.data.Profit / totalProfit * 100).toFixed(2)}%
                    `);
            })
            .on("mouseout", (event) => {
                d3.select(event.currentTarget)
                    .transition()
                    .duration(200)
                    .attr("d", arcGenerator);

                tooltip.style("display", "none");
            });

        // Add Labels
        svg.selectAll("text")
            .data(data_ready)
            .enter()
            .append("text")
            .text(d => `${d.data.Region} (${((d.data.Profit / totalProfit) * 100).toFixed(1)}%)`)
            .attr("transform", d => `translate(${arcGenerator.centroid(d)})`)
            .style("text-anchor", "middle")
            .style("font-size", "14px")
            .style("font-weight", "bold")
            .style("fill", "#333");
    }
</script>

<style>
    /* General Chart Container */
    .chart-container {
        display: flex;
        flex-direction: column;
        align-items: center;
        padding: 1rem;
        margin: auto;
        border-radius: 10px;
        background: white;
        box-shadow: 0px 4px 8px rgba(0, 0, 0, 0.1);
        max-width: 600px;
    }

    /* Filter Section Styling */
    /* .filter-container {
        display: flex;
        justify-content: center;
        align-items: center;
        gap: 1rem;
        width: 100%;
        padding: 1rem;
        background: #f3f4f6;
        border-radius: 8px;
        box-shadow: 0px 4px 8px rgba(0, 0, 0, 0.1);
    }

    label {
        font-size: 1.1rem;
        font-weight: bold;
        color: #333;
    }

    select {
        padding: 0.5rem;
        font-size: 1rem;
        border: 2px solid #3498db;
        border-radius: 8px;
        background: #f9f9f9;
        cursor: pointer;
    } */

    /* Pie Chart Styling */
    #pieChart {
        width: 100%;
        max-width: 100%;
        height: auto;
        padding: 1rem;
        overflow: hidden;
        box-sizing: border-box;
    }

    /* Responsive Adjustments */
    /* @media (max-width: 768px) {
        .filter-container {
            flex-direction: column;
        }

        select {
            width: 100%;
        }
    } */
</style>

<div class="chart-container">
    <!-- Filter Section -->
    <!-- <section class="filter-container">
        <label for="segmentSelect">Select Customer Segment:</label>
        <select id="segmentSelect" bind:value={selectedSegment} on:change={filterData}>
            <option value="All Segments">All Segments</option>
            {#each uniqueSegments as segment}
                <option value={segment}>{segment}</option>
            {/each}
        </select>
    </section> -->

    <!-- Pie Chart Section -->
    <div id="pieChart"></div>
</div>
