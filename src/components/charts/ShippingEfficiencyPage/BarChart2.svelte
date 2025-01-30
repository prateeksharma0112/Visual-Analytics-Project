<script>
    import { onMount } from 'svelte';
    import * as d3 from 'd3';

    export let data = [];
    export let xField; // Example: 'Region'
    export let yField; // Example: 'Avg Fulfillment Time'

    let container;

    $: if (data && Array.isArray(data) && data.length > 0 && container) {
        console.log("📊 Processing Fulfillment Time Data:", data);
        drawBarChart();
    }

    function drawBarChart() {
        if (!container) {
            console.error("🚨 BarChart Error: Container not found.");
            return;
        }

        if (!Array.isArray(data) || data.length === 0 || !xField || !yField) {
            d3.select(container).html('<p style="color: red; text-align: center;">No valid data for Bar Chart</p>');
            console.error("🚨 BarChart Error: Data is empty or invalid.");
            return;
        }

        // ✅ Chart Dimensions
        const margin = { top: 30, right: 30, bottom: 70, left: 70 };
        const width = 500 - margin.left - margin.right;
        const height = 350 - margin.top - margin.bottom;

        // ✅ Clear & Create SVG
        const svg = d3.select(container)
            .html('')
            .append('svg')
            .attr('width', width + margin.left + margin.right)
            .attr('height', height + margin.top + margin.bottom)
            .append('g')
            .attr('transform', `translate(${margin.left}, ${margin.top})`);

        // ✅ Tooltip (Corrected Logic)
        const tooltip = d3.select("body")
            .append("div")
            .attr("class", "tooltip")
            .style("position", "absolute")
            .style("background", "#fff")
            .style("border", "1px solid #ccc")
            .style("padding", "8px")
            .style("border-radius", "5px")
            .style("box-shadow", "0px 4px 8px rgba(0,0,0,0.1)")
            .style("font-size", "14px")
            .style("color", "#333")
            .style("pointer-events", "none")
            .style("opacity", 0);

        // ✅ Scales
        const xScale = d3.scaleBand()
            .domain(data.map(d => d[xField]))
            .range([0, width])
            .padding(0.5);

        const yScale = d3.scaleLinear()
            .domain([0, d3.max(data, d => d[yField]) || 1])
            .nice()
            .range([height, 0]);

        // ✅ X & Y Axes
        svg.append("g")
            .attr("transform", `translate(0, ${height})`)
            .call(d3.axisBottom(xScale))
            .selectAll("text")
            .style("text-anchor", "end")
            .attr("dx", "-.8em")
            .attr("dy", ".15em")
            .attr("transform", "rotate(-30)")
            .style("font-size", "12px");

        svg.append("g").call(d3.axisLeft(yScale).ticks(6))
            .style("font-size", "12px");

        // ✅ X & Y Labels
        svg.append("text")
            .attr("x", width / 2)
            .attr("y", height + 60)
            .attr("text-anchor", "middle")
            .style("font-size", "14px")
            .style("fill", "#2C3E50")
            .text("Shipping Mode");

        svg.append("text")
            .attr("x", -height / 2)
            .attr("y", -45)
            .attr("transform", "rotate(-90)")
            .attr("text-anchor", "middle")
            .style("font-size", "14px")
            .style("fill", "#2C3E50")
            .text("Fulfillment Time (Days)");

        // ✅ Bars with Tooltip
        svg.selectAll(".bar")
            .data(data)
            .enter()
            .append("rect")
            .attr("x", d => xScale(d[xField]))
            .attr("y", d => yScale(d[yField]))
            .attr("width", xScale.bandwidth())
            .attr("height", d => height - yScale(d[yField]))
            .attr("fill", "#0077cc")
            .on("mouseover", (event, d) => {
                d3.select(event.target)
                    .attr("fill", "#0056a3");

                tooltip.transition().duration(200).style("opacity", 1);
                tooltip.html(`
                    <strong>${xField}:</strong> ${d[xField]}<br>
                    <strong>${yField}:</strong> ${d[yField]} days
                `)
                    .style("left", `${event.pageX + 10}px`)
                    .style("top", `${event.pageY - 20}px`);
            })
            .on("mousemove", (event) => {
                tooltip.style("left", `${event.pageX + 10}px`)
                    .style("top", `${event.pageY - 20}px`);
            })
            .on("mouseout", (event, d) => {
                d3.select(event.target)
                    .attr("fill", "#0077cc");
                tooltip.transition().duration(200).style("opacity", 0);
            });

        // ✅ Labels on Bars
        svg.selectAll(".label")
            .data(data)
            .enter()
            .append("text")
            .attr("x", d => xScale(d[xField]) + xScale.bandwidth() / 2)
            .attr("y", d => yScale(d[yField]) - 5)
            .attr("text-anchor", "middle")
            .style("font-size", "12px")
            .style("fill", "#2C3E50")
            .text(d => d[yField]);
    }
</script>

<div bind:this={container}></div>
