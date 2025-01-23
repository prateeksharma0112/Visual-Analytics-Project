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
        drawHeatmap();
    }

    onMount(() => {
        uniqueSegments = [...new Set(data.map(d => d.Segment))];
        filteredData = [...data];
        drawHeatmap();
        window.addEventListener("resize", drawHeatmap); // Redraw heatmap on window resize
    });

    function drawHeatmap() {
        const container = document.querySelector("#heatmap");
        const containerWidth = container.clientWidth;
        const containerHeight = 500;

        const margin = { top: 50, right: 30, bottom: 60, left: 100 };
        const width = containerWidth - margin.left - margin.right;
        const height = containerHeight - margin.top - margin.bottom;

        if (filteredData.length === 0) {
            d3.select("#heatmap").html("<p>No data available for the selected segment.</p>");
            return;
        }

        // Aggregate data
        const aggregatedData = d3.rollups(
            filteredData,
            v => ({
                totalSales: d3.sum(v, d => d.Sales),
                totalProfit: d3.sum(v, d => d.Profit),
            }),
            d => Math.floor(d.Discount * 100 / 10) * 10, // Group by discount range
            d => Math.floor(d.Sales / 1000) * 1000 // Group by sales range
        );

        const dataForHeatmap = aggregatedData.flatMap(([discountBin, salesBins]) =>
            salesBins.map(([salesBin, values]) => ({
                DiscountBin: discountBin,
                SalesBin: salesBin,
                TotalSales: values.totalSales,
                TotalProfit: values.totalProfit,
            }))
        );

        const sortedDiscountBins = [...new Set(dataForHeatmap.map(d => d.DiscountBin))].sort((a, b) => a - b);
        const sortedSalesBins = [...new Set(dataForHeatmap.map(d => d.SalesBin))].sort((a, b) => a - b);

        const svg = d3.select("#heatmap")
            .html("")
            .append("svg")
            .attr("width", containerWidth)
            .attr("height", containerHeight)
            .append("g")
            .attr("transform", `translate(${margin.left}, ${margin.top})`);

        const xScale = d3.scaleBand()
            .domain(sortedDiscountBins)
            .range([0, width])
            .padding(0.05);

        const yScale = d3.scaleBand()
            .domain(sortedSalesBins)
            .range([height, 0])
            .padding(0.05);

        const colorScale = d3.scaleSequential(d3.interpolateBlues)
            .domain([0, d3.max(dataForHeatmap, d => d.TotalSales)]);

        // Draw heatmap rectangles
        svg.selectAll("rect")
            .data(dataForHeatmap)
            .enter()
            .append("rect")
            .attr("x", d => xScale(d.DiscountBin))
            .attr("y", d => yScale(d.SalesBin))
            .attr("width", xScale.bandwidth())
            .attr("height", yScale.bandwidth())
            .attr("fill", d => colorScale(d.TotalSales || 0))
            .style("stroke", "#ccc")
            .on("mouseover", (event, d) => {
                const tooltip = d3.select("#heatmapTooltip");
                tooltip.style("display", "block")
                    .html(`
                        <strong>Discount Range:</strong> ${d.DiscountBin}-${d.DiscountBin + 10}%<br>
                        <strong>Sales Range:</strong> $${d.SalesBin}-$${d.SalesBin + 1000}<br>
                        <strong>Total Sales:</strong> $${d.TotalSales.toFixed(2)}<br>
                        <strong>Total Profit:</strong> $${d.TotalProfit.toFixed(2)}
                    `)
                    .style("left", `${event.pageX + 10}px`)
                    .style("top", `${event.pageY - 20}px`);
            })
            .on("mouseout", () => d3.select("#heatmapTooltip").style("display", "none"));

        // Add X-axis
        svg.append("g")
            .attr("transform", `translate(0, ${height})`)
            .call(d3.axisBottom(xScale).tickFormat(d => `${d}-${d + 10}%`))
            .append("text")
            .attr("x", width / 2)
            .attr("y", 50)
            .attr("fill", "black")
            .style("font-size", "14px")
            .style("font-weight", "bold")
            .text("Discount (%)");

        // Add Y-axis
        svg.append("g")
            .call(d3.axisLeft(yScale).tickFormat(d => `$${(d / 1000).toFixed(0)}k`))
            .append("text")
            .attr("x", -height / 2)
            .attr("y", -70)
            .attr("transform", "rotate(-90)")
            .attr("fill", "black")
            .style("font-size", "14px")
            .style("font-weight", "bold")
            .text("Sales Range (USD)");
    }
</script>

<!-- ✅ Stylish Segment Filter Section -->
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
    <label for="segmentSelect" style="font-size: 1.1rem; font-weight: bold;">Select Customer Segment:</label>
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

<!-- Heatmap Container -->
<div id="heatmap" style="
    width: 100%; 
    max-width: 800px; 
    height: 500px; 
    margin: auto; 
    border-radius: 8px; 
    box-shadow: 0px 4px 8px rgba(0, 0, 0, 0.1); 
    background-color: #ffffff;">
</div>

<!-- Tooltip -->
<div id="heatmapTooltip" style="
    position: absolute;
    background: white;
    border: 1px solid #ccc;
    border-radius: 5px;
    padding: 10px;
    display: none;
    pointer-events: none;
    box-shadow: 0px 4px 8px rgba(0, 0, 0, 0.1);
"></div>
