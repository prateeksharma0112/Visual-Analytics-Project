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
        const width = document.getElementById("barChart").clientWidth - margin.left - margin.right;
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

        // Draw Bars with the Gradient Color
        svg.selectAll("rect")
            .data(salesByCategory)
            .enter()
            .append("rect")
            .attr("x", d => xScale(d[0]))
            .attr("y", height)
            .attr("width", xScale.bandwidth())
            .attr("height", 0)
            .style("fill", "url(#barGradient)")
            .attr("rx", 8) // Rounded corners
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

<style>
    /* Container for Bar Chart and Filter Section */
    .chart-container {
        display: flex;
        flex-direction: column;
        align-items: center;
        padding: 1rem;
        border-radius: 10px;
        background-color: #ffffff;
        box-shadow: 0px 4px 8px rgba(0, 0, 0, 0.1);
        max-width: 800px;
        margin: auto;
    }

    /* Filter Section Styling */
    .select-container {
        display: flex;
        justify-content: center;
        align-items: center;
        gap: 1rem;
        width: 100%;
        padding: 1rem;
        background: #f9f9f9;
        border-radius: 8px;
        box-shadow: 0px 4px 8px rgba(0, 0, 0, 0.1);
        margin-bottom: 1rem;
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
    }

    /* Chart Area Styling */
    #barChart {
        width: 100%; /* Full width of the container */
        max-width: 100%; /* Prevent overflow */
        height: 400px; /* Fixed height for the chart */
        padding: 1rem;
        overflow: hidden; /* Prevent unnecessary overflow */
        box-sizing: border-box;
    }

    /* Responsive Adjustments */
    @media (max-width: 768px) {
        .chart-container {
            padding: 0.5rem;
        }

        .select-container {
            flex-direction: column;
            gap: 0.5rem;
        }

        select {
            width: 100%;
        }
    }
</style>

<div class="chart-container">
    <!-- Filter Section -->
    <section class="select-container">
        <label for="region">Select Region:</label>
        <select bind:value={selectedRegion} on:change={filterData}>
            <option value="All Regions">All Regions</option>
            {#each uniqueRegions as region}
                <option value={region}>{region}</option>
            {/each}
        </select>
    </section>

    <!-- Bar Chart Section -->
    <div id="barChart"></div>
</div>
