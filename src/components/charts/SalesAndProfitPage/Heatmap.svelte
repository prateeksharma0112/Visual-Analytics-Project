<script>
    import { onMount } from 'svelte';
    import * as d3 from 'd3';

    export let data = [];
    let svg;
    let width = 800; // Default width
    let height = 400; // Default height
    let margin = { top: 10, right: 10, bottom: 40, left: 120 };

    let heatmapData = [];
    let xScale, yScale, colorScale;

    function processHeatmapData() {
        heatmapData = d3.rollups(
            data,
            (v) => ({
                sales: d3.sum(v, (d) => parseFloat(d.Sales) || 0),
                profit: d3.sum(v, (d) => parseFloat(d.Profit) || 0),
            }),
            (d) => d.Region,
            (d) => d.State
        ).flatMap(([region, states]) =>
            states.map(([state, metrics]) => ({
                region,
                state,
                sales: metrics.sales,
                profit: metrics.profit,
            }))
        );
    }

    function createScales() {
        const xDomain = [...new Set(heatmapData.map((d) => d.region))];
        const yDomain = [...new Set(heatmapData.map((d) => d.state))];

        xScale = d3.scaleBand()
            .domain(xDomain)
            .range([margin.left, width - margin.right])
            .padding(0.2);

        yScale = d3.scaleBand()
            .domain(yDomain)
            .range([margin.top, height - margin.bottom])
            .padding(0.2);

        colorScale = d3.scaleSequential(d3.interpolateBlues)
            .domain([0, d3.max(heatmapData, (d) => d.sales)]);
    }

    function drawHeatmap() {
        const svgElement = d3.select(svg);
        svgElement.selectAll("*").remove();

        // Draw rectangles
        svgElement.selectAll("rect")
            .data(heatmapData)
            .join("rect")
            .attr("x", (d) => xScale(d.region))
            .attr("y", (d) => yScale(d.state))
            .attr("width", xScale.bandwidth())
            .attr("height", yScale.bandwidth())
            .attr("fill", (d) => colorScale(d.sales))
            .attr("stroke", "#fff")
            .attr("stroke-width", 1)
            .on("mouseover", function (event, d) {
                d3.select(this).attr("stroke-width", 2);
                showTooltip(event, d);
            })
            .on("mouseout", function () {
                d3.select(this).attr("stroke-width", 1);
                hideTooltip();
            });

        // Add X-axis
        svgElement.append("g")
            .attr("transform", `translate(0,${height - margin.bottom})`)
            .call(d3.axisBottom(xScale))
            .selectAll("text")
            .attr("transform", "rotate(-45)")
            .style("text-anchor", "end");

        // Add Y-axis
        svgElement.append("g")
            .attr("transform", `translate(${margin.left},0)`)
            .call(d3.axisLeft(yScale));
        
        // Add X-axis label
        svgElement.append("text")
            .attr("x", (width - margin.left - margin.right) / 2 + margin.left)
            .attr("y", height - 5) // Below the axis
            .attr("text-anchor", "middle")
            .style("font-size", "14px")
            .style("font-weight", "bold")
            .text("Region");

        // Add Y-axis label
        svgElement.append("text")
            .attr("transform", "rotate(-90)")
            .attr("x", -(height - margin.top - margin.bottom) / 2 - margin.top)
            .attr("y", 20) // Left of the axis
            .attr("text-anchor", "middle")
            .style("font-size", "14px")
            .style("font-weight", "bold")
            .text("States");
    }

    let tooltip;

    function createTooltip() {
        tooltip = d3.select("body")
            .append("div")
            .style("position", "absolute")
            .style("background", "#fff")
            .style("border", "1px solid #ccc")
            .style("border-radius", "4px")
            .style("padding", "8px")
            .style("box-shadow", "0 4px 8px rgba(0,0,0,0.1)")
            .style("visibility", "hidden");
    }

    function showTooltip(event, d) {
        tooltip.style("visibility", "visible")
            .style("top", `${event.pageY + 10}px`)
            .style("left", `${event.pageX + 10}px`)
            .html(`
                <strong>Region:</strong> ${d.region}<br>
                <strong>State:</strong> ${d.state}<br>
                <strong>Sales:</strong> $${d.sales.toFixed(2)}<br>
                <strong>Profit:</strong> $${d.profit.toFixed(2)}
            `);
    }

    function hideTooltip() {
        tooltip.style("visibility", "hidden");
    }

    // Handle window resize for responsiveness
    function handleResize() {
        const container = svg.parentElement;
        width = container.clientWidth;
        height = container.clientHeight;
        createScales();
        drawHeatmap();
    }

    onMount(() => {
        processHeatmapData();
        createTooltip();
        handleResize();
        window.addEventListener("resize", handleResize);
    });

    $: if (data.length) {
        processHeatmapData();
        createScales();
        drawHeatmap();
    }
</script>

<svg bind:this={svg} viewBox={`0 0 ${width} ${height}`} preserveAspectRatio="xMidYMid meet"></svg>

<style>
    svg {
        width: 100%;
        height: auto;
    }
</style>
