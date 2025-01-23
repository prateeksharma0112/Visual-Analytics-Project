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
        uniqueRegions = [...new Set(data.map(d => d.Region))];
        filteredData = [...data];
        drawLineChart();
    });

    function drawLineChart() {
        if (filteredData.length === 0) {
            d3.select("#lineChart").html("<p>No data available for the selected region.</p>");
            return;
        }

        const profitTrends = d3.rollups(
            filteredData,
            v => d3.sum(v, d => d.Profit),
            d => d3.timeFormat("%Y-%m")(new Date(d["Order Date"]))
        ).sort((a, b) => new Date(a[0]) - new Date(b[0]));

        const container = document.getElementById("lineChart");
        const margin = { top: 50, right: 30, bottom: 60, left: 80 };
        const width = container.clientWidth - margin.left - margin.right;
        const height = 400 - margin.top - margin.bottom;

        const svg = d3.select(container)
            .html('')
            .append("svg")
            .attr("width", width + margin.left + margin.right)
            .attr("height", height + margin.top + margin.bottom)
            .append("g")
            .attr("transform", `translate(${margin.left}, ${margin.top})`);

        const xScale = d3.scaleTime()
            .domain(d3.extent(profitTrends, d => new Date(d[0])))
            .range([0, width]);

        const yScale = d3.scaleLinear()
            .domain([0, d3.max(profitTrends, d => d[1])])
            .range([height, 0]);

        const line = d3.line()
            .x(d => xScale(new Date(d[0])))
            .y(d => yScale(d[1]))
            .curve(d3.curveCatmullRom);

        const gradient = svg.append("defs")
            .append("linearGradient")
            .attr("id", "lineGradient")
            .attr("x1", "0%")
            .attr("y1", "0%")
            .attr("x2", "100%")
            .attr("y2", "0%");

        gradient.append("stop")
            .attr("offset", "0%")
            .attr("stop-color", "#4facfe");

        gradient.append("stop")
            .attr("offset", "100%")
            .attr("stop-color", "#00f2fe");

        svg.append("path")
            .datum(profitTrends)
            .attr("fill", "none")
            .attr("stroke", "url(#lineGradient)")
            .attr("stroke-width", 3)
            .attr("d", line);

        svg.selectAll("circle")
            .data(profitTrends)
            .enter()
            .append("circle")
            .attr("cx", d => xScale(new Date(d[0])))
            .attr("cy", d => yScale(d[1]))
            .attr("r", 5)
            .attr("fill", "#4facfe");

        const tooltip = d3.select(container)
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

        svg.append("g")
            .attr("transform", `translate(0, ${height})`)
            .call(d3.axisBottom(xScale).ticks(d3.timeYear.every(1)).tickFormat(d3.timeFormat("%Y")));

        svg.append("g").call(d3.axisLeft(yScale));

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

<style>
    /* General Chart Styling */
    .chart-container {
        display: flex;
        flex-direction: column;
        align-items: center;
        padding: 1rem;
        margin: auto;
        border-radius: 10px;
        background: white;
        box-shadow: 0px 4px 8px rgba(0, 0, 0, 0.1);
        max-width: 800px;
    }

    /* Filter Section Styling */
    .filter-container {
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
    }

    /* Line Chart Styling */
    #lineChart {
        width: 100%;
        height: 100%;
        max-width: 100%;
        padding: 1rem;
        overflow: hidden;
    }

    /* Responsive Adjustments */
    @media (max-width: 768px) {
        .filter-container {
            flex-direction: column;
        }

        select {
            width: 100%;
        }
    }
</style>

<div class="chart-container">
    <!-- Filter Section -->
    <section class="filter-container">
        <label for="region">Select Region:</label>
        <select bind:value={selectedRegion} on:change={filterData}>
            <option value="All Regions">All Regions</option>
            {#each uniqueRegions as region}
                <option value={region}>{region}</option>
            {/each}
        </select>
    </section>

    <!-- Line Chart -->
    <div id="lineChart"></div>
</div>
