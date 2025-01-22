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
            .range(d3.schemeCategory10);

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
            .style("opacity", 0.8)
            .on("mouseover", (event, d) => {
                // Highlight the slice
                d3.select(event.currentTarget)
                    .transition()
                    .duration(200)
                    .attr("d", arcHover);

                // Show tooltip near the slice corner
                const [x, y] = arcHover.centroid(d); // Get slice corner position
                tooltip.style("display", "block")
                    .style("left", `${x + width / 2}px`)
                    .style("top", `${y + height / 2}px`)
                    .html(`
                        <strong>Region:</strong> ${d.data.Region}<br>
                        <strong>Profit:</strong> $${d.data.Profit.toFixed(2)}<br>
                        <strong>Contribution:</strong> ${(d.data.Profit / totalProfit * 100).toFixed(2)}%
                    `);
            })
            .on("mouseout", (event) => {
                // Reset slice size
                d3.select(event.currentTarget)
                    .transition()
                    .duration(200)
                    .attr("d", arcGenerator);

                // Hide tooltip
                tooltip.style("display", "none");
            });

        // Add Labels
        svg.selectAll("text")
            .data(data_ready)
            .enter()
            .append("text")
            .text(d => `${d.data.Region}`)
            .attr("transform", d => `translate(${arcGenerator.centroid(d)})`)
            .style("text-anchor", "middle")
            .style("font-size", "12px")
            .style("font-weight", "bold")
            .style("fill", "#000");
    }
</script>

<!-- ✅ Segment Filter Section -->
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
    <label for="segmentSelect" style="font-size: 1.1rem; font-weight: bold;">Select Segment:</label>
    <select id="segmentSelect" bind:value={selectedSegment} on:change={filterData} style="
        padding: 0.5rem;
        border: 2px solid #3498db;
        border-radius: 8px;
        font-size: 1rem;
        background: #f9f9f9;
    ">
        <option value="All Segments">All Segments</option>
        {#each uniqueSegments as segment}
            <option value={segment}>{segment}</option>
        {/each}
    </select>
</section>

<!-- ✅ Pie Chart Container -->
<div id="pieChart" style="padding: 1rem; position: relative;"></div>
