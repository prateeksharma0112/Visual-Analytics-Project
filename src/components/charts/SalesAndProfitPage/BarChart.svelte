<script>
    import { onMount } from 'svelte';
    import * as d3 from 'd3';

    export let data = [];
    export let xField = 'category';
    export let yFields = ['sales', 'profit'];

    let svg;
    let width = 800;
    let height = 400;
    let margin = { top: 10, right: 20, bottom: 100, left: 80 };

    let chartData = [];
    let xScale, yScale, colorScale;

    function processData() {
        chartData = d3.rollups(
            data,
            (v) => ({
                sales: d3.sum(v, (d) => parseFloat(d.Sales) || 0),
                profit: d3.sum(v, (d) => parseFloat(d.Profit) || 0),
            }),
            (d) => d[xField]
        ).map(([key, value]) => ({ [xField]: key, ...value }));
    }

    function createScales() {
        xScale = d3.scaleBand()
            .domain(chartData.map((d) => d[xField]))
            .range([margin.left, width - margin.right])
            .padding(0.2);

        yScale = d3.scaleLinear()
            .domain([0, d3.max(chartData, (d) => Math.max(d.sales, d.profit))])
            .range([height - margin.bottom, margin.top])
            .nice();

        colorScale = d3.scaleOrdinal().domain(yFields).range(['#4facfe', '#f76b1c']);
    }

    function drawBarChart() {
        const svgElement = d3.select(svg);

        // Clear existing content
        svgElement.selectAll("*").remove();

        // Add bars
        yFields.forEach((field, i) => {
            svgElement
                .selectAll(`.bar-${field}`)
                .data(chartData)
                .join("rect")
                .attr("class", `bar-${field}`)
                .attr("x", (d) => xScale(d[xField]) + (xScale.bandwidth() / yFields.length) * i)
                .attr("y", (d) => yScale(d[field]))
                .attr("width", xScale.bandwidth() / yFields.length - 5) // Add spacing between bars
                .attr("height", (d) => height - margin.bottom - yScale(d[field]))
                .attr("fill", colorScale(field))
                .on("mouseover", function (event, d) {
                    d3.select(this).attr("opacity", 0.8);
                    showTooltip(event, d, field);
                })
                .on("mouseout", function () {
                    d3.select(this).attr("opacity", 1);
                    hideTooltip();
                });
        });

        // Add X-axis
        svgElement
            .append("g")
            .attr("transform", `translate(0,${height - margin.bottom})`)
            .call(d3.axisBottom(xScale))
            .selectAll("text")
            .attr("transform", "rotate(-45)")
            .style("text-anchor", "end")
            .style("font-size", "12px");

        // Add Y-axis
        svgElement
            .append("g")
            .attr("transform", `translate(${margin.left},0)`)
            .call(d3.axisLeft(yScale).ticks(6))
            .selectAll("text")
            .style("font-size", "12px");

        // Add X-axis label
        svgElement
            .append("text")
            .attr("x", width / 2)
            .attr("y", height - 15)
            .attr("text-anchor", "middle")
            .text(xField.charAt(0).toUpperCase() + xField.slice(1))
            .style("font-size", "16px")
            .style("font-weight", "bold");

        // Add Y-axis label
        svgElement
            .append("text")
            .attr("x", -(height / 2))
            .attr("y", 20)
            .attr("transform", "rotate(-90)")
            .attr("text-anchor", "middle")
            .text("Amount ($)")
            .style("font-size", "16px")
            .style("font-weight", "bold");

        // Add legend
        const legend = svgElement
            .selectAll(".legend")
            .data(yFields)
            .join("g")
            .attr("class", "legend")
            .attr("transform", (_, i) => `translate(${width - margin.right - 100},${margin.top + i * 20})`);

        legend
            .append("rect")
            .attr("width", 16)
            .attr("height", 16)
            .attr("fill", (d) => colorScale(d));

        legend
            .append("text")
            .attr("x", 20)
            .attr("y", 12)
            .text((d) => d.charAt(0).toUpperCase() + d.slice(1))
            .style("font-size", "12px")
            .style("fill", "#333");
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

    function showTooltip(event, d, field) {
        tooltip.style("visibility", "visible")
            .style("top", `${event.pageY + 10}px`)
            .style("left", `${event.pageX + 10}px`)
            .html(`
                <strong>${xField}:</strong> ${d[xField]}<br>
                <strong>${field.charAt(0).toUpperCase() + field.slice(1)}:</strong> $${d[field].toFixed(2)}
            `);
    }

    function hideTooltip() {
        tooltip.style("visibility", "hidden");
    }

    onMount(() => {
        processData();
        createScales();
        createTooltip();
        drawBarChart();
    });

    $: if (data.length) {
        processData();
        createScales();
        drawBarChart();
    }
</script>

<svg bind:this={svg} viewBox={`0 0 ${width} ${height}`} preserveAspectRatio="xMidYMid meet"></svg>

<style>
    svg {
        display: block;
        margin: auto;
        width: 100%;
        height: auto;
    }
</style>
