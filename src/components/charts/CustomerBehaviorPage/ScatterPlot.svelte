<script>
    import * as d3 from 'd3';
    import { onMount } from 'svelte';

    export let data = []; // Expected: [{ value: Sales, discount: Discount }]
    export let xField = "value"; // Default: Sales
    export let yField = "discount"; // Default: Discount (%)

    let container;

    // Debugging
    $: {
        if (data && Array.isArray(data) && data.length > 0) {
            console.log("📊 ScatterPlot Data:", data);
            drawScatterPlot();
        } else {
            console.warn("🚨 ScatterPlot Warning: No valid data received!");
        }
    }

    function drawScatterPlot() {
        if (!container) {
            console.error("🚨 ScatterPlot Error: Container not found.");
            return;
        }

        console.log("📈 ScatterPlot Debug Data:", { data, xField, yField });

        if (!Array.isArray(data) || data.length === 0 || !xField || !yField) {
            d3.select(container).html('<p style="color: red; text-align: center;">No valid data for Scatter Plot</p>');
            console.error("🚨 ScatterPlot Error: Data is empty or invalid.");
            return;
        }

        // ✅ Define chart dimensions
        const margin = { top: 20, right: 30, bottom: 50, left: 50 };
        const width = 500 - margin.left - margin.right;
        const height = 300 - margin.top - margin.bottom;

        // ✅ Append SVG
        const svg = d3.select(container)
            .html('')
            .append("svg")
            .attr("width", width + margin.left + margin.right)
            .attr("height", height + margin.top + margin.bottom)
            .append("g")
            .attr("transform", `translate(${margin.left}, ${margin.top})`);

        // ✅ X Scale (Order Value / Sales)
        const xScale = d3.scaleLinear()
            .domain([0, d3.max(data, d => d[xField])]) // Min: 0, Max: Highest Sales Value
            .nice()
            .range([0, width]);

        // ✅ Y Scale (Discount %)
        const yScale = d3.scaleLinear()
            .domain([0, d3.max(data, d => d[yField])]) // Min: 0, Max: Highest Discount %
            .nice()
            .range([height, 0]);

        // ✅ X Axis
        svg.append("g")
            .attr("transform", `translate(0, ${height})`)
            .call(d3.axisBottom(xScale).ticks(5).tickFormat(d => `$${d}`));

        // ✅ Y Axis
        svg.append("g")
            .call(d3.axisLeft(yScale).ticks(5).tickFormat(d => `${d}%`));

        // ✅ Tooltip
        const tooltip = d3.select(container)
            .append("div")
            .style("position", "absolute")
            .style("background", "#fff")
            .style("border", "1px solid #ccc")
            .style("padding", "5px")
            .style("border-radius", "5px")
            .style("box-shadow", "0px 4px 8px rgba(0,0,0,0.1)")
            .style("display", "none");

        // ✅ Add Points
        svg.selectAll("circle")
            .data(data)
            .enter()
            .append("circle")
            .attr("cx", d => xScale(d[xField]))
            .attr("cy", d => yScale(d[yField]))
            .attr("r", 5)
            .attr("fill", "#4facfe")
            .style("opacity", 0.7)
            .on("mouseover", (event, d) => {
                tooltip.style("display", "block")
                    .html(`<strong>Order Value:</strong> $${d[xField].toFixed(2)}<br><strong>Discount:</strong> ${d[yField].toFixed(2)}%`)
                    .style("left", `${event.pageX + 10}px`)
                    .style("top", `${event.pageY - 20}px`);
            })
            .on("mouseout", () => tooltip.style("display", "none"));

        // ✅ Chart Labels
        svg.append("text")
            .attr("x", width / 2)
            .attr("y", height + 40)
            .attr("text-anchor", "middle")
            .style("font-size", "14px")
            .text("Order Value ($)");

        svg.append("text")
            .attr("transform", "rotate(-90)")
            .attr("y", -40)
            .attr("x", -height / 2)
            .attr("text-anchor", "middle")
            .style("font-size", "14px")
            .text("Discount (%)");

        console.log("✅ Scatter Plot Rendered Successfully!");
    }

    onMount(() => {
        if (container) {
            console.log("✅ ScatterPlot Container Mounted Successfully.");
            drawScatterPlot();
        } else {
            console.warn("🚨 ScatterPlot Container Not Found. Will Retry.");
        }
    });
</script>

<div bind:this={container}></div>
