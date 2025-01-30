<script>
    import { onMount } from "svelte";
    import * as d3 from "d3";

    export let data = []; // Pre-binned data with 'days' and 'count'

    let container;

    // Ensure the chart only renders when data is available & container exists
    $: if (data && data.length > 0 && container) {
        console.log("📊 Fulfillment Time Distribution Data:", data);
        drawHistogram();
    }

    function drawHistogram() {
        if (!container) {
            console.error("🚨 Histogram Error: Container not found.");
            return;
        }

        console.log("📈 Debug Histogram Data:", data);

        // ✅ Sort Data by Days
        data.sort((a, b) => a.days - b.days);

        // ✅ Chart Dimensions
        const margin = { top: 30, right: 30, bottom: 50, left: 60 };
        const width = 500 - margin.left - margin.right;
        const height = 350 - margin.top - margin.bottom;

        // ✅ Clear & Create SVG
        const svg = d3.select(container)
            .html("") // Clear previous chart
            .append("svg")
            .attr("width", width + margin.left + margin.right)
            .attr("height", height + margin.top + margin.bottom)
            .append("g")
            .attr("transform", `translate(${margin.left}, ${margin.top})`);

        // ✅ Tooltip
        const tooltip = d3.select("body")
            .append("div")
            .attr("class", "tooltip")
            .style("position", "absolute")
            .style("background", "#fff")
            .style("border", "1px solid #ccc")
            .style("padding", "8px")
            .style("border-radius", "5px")
            .style("box-shadow", "0px 4px 8px rgba(0,0,0,0.1)")
            .style("pointer-events", "none")
            .style("opacity", 0);

        // ✅ X & Y Scales
        const xScale = d3.scaleLinear()
            .domain([0, d3.max(data, d => d.days) + 1]) // Extend domain to include the last bin
            .range([0, width]);

        const yScale = d3.scaleLinear()
            .domain([0, d3.max(data, d => d.count)])
            .nice()
            .range([height, 0]);

        const binWidth = xScale(1) - xScale(0); // Calculate width based on scale

        // ✅ Axes
        svg.append("g")
            .attr("transform", `translate(0, ${height})`)
            .call(d3.axisBottom(xScale).ticks(data.length).tickFormat(d => `${d} days`))
            .selectAll("text")
            .attr("transform", "rotate(-30)")
            .style("text-anchor", "end")
            .style("font-size", "12px");

        svg.append("g")
            .call(d3.axisLeft(yScale).ticks(6))
            .style("font-size", "12px");

        // ✅ X & Y Labels
        svg.append("text")
            .attr("x", width / 2)
            .attr("y", height + 48)
            .attr("text-anchor", "middle")
            .style("font-size", "14px")
            .style("fill", "#333")
            .text("Fulfillment Time (days)");

        svg.append("text")
            .attr("x", -height / 2)
            .attr("y", -45)
            .attr("transform", "rotate(-90)")
            .attr("text-anchor", "middle")
            .style("font-size", "14px")
            .style("fill", "#333")
            .text("Number of Orders");

        // ✅ Bars with Tooltip
        svg.selectAll(".bar")
            .data(data)
            .enter()
            .append("rect")
            .attr("x", d => xScale(d.days))
            .attr("width", binWidth - 2) // Narrow width for proper alignment
            .attr("y", d => yScale(d.count))
            .attr("height", d => height - yScale(d.count))
            .attr("fill", "#0077cc")
            .on("mouseover", (event, d) => {
                d3.select(event.target).attr("fill", "#0056a3"); // Highlight color
                tooltip.transition().duration(200).style("opacity", 1);
                tooltip.html(`
                    <strong>Days:</strong> ${d.days}<br>
                    <strong>Orders:</strong> ${d.count}
                `)
                .style("left", `${event.pageX + 10}px`)
                .style("top", `${event.pageY - 20}px`);
            })
            .on("mousemove", (event) => {
                tooltip.style("left", `${event.pageX + 10}px`)
                    .style("top", `${event.pageY - 20}px`);
            })
            .on("mouseout", (event, d) => {
                d3.select(event.target).attr("fill", "#0077cc"); // Restore color
                tooltip.transition().duration(200).style("opacity", 0);
            });

        // ✅ Labels on Bars
        svg.selectAll(".label")
            .data(data)
            .enter()
            .append("text")
            .attr("x", d => xScale(d.days) + binWidth / 2)
            .attr("y", d => yScale(d.count) - 5)
            .attr("text-anchor", "middle")
            .style("font-size", "12px")
            .style("fill", "#333")
            .text(d => d.count);
    }
</script>

<div bind:this={container}></div>
