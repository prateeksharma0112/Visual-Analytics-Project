<script>
    import { onMount } from 'svelte';
    import * as d3 from 'd3';

    export let data = [];
    export let labelField;
    export let valueField;

    let container;

    $: {
        if (data && data.length > 0) {
            console.log("📊 PieChart Received Data:", data);
            drawPieChart();
        } else {
            console.warn("🚨 No valid data received for PieChart!");
        }
    }

    function drawPieChart() {
        // ✅ Chart Dimensions
        const width = 400, height = 300;
        const radius = Math.min(width, height) / 2;
        const color = d3.scaleOrdinal(d3.schemeCategory10);

        // ✅ Clear previous chart
        d3.select(container).html('');

        // ✅ Create SVG container
        const svg = d3.select(container)
            .append('svg')
            .attr('width', width)
            .attr('height', height)
            .append('g')
            .attr('transform', `translate(${width / 2}, ${height / 2})`);

        // ✅ Create pie layout
        const pie = d3.pie()
            .sort(null)
            .value(d => d[valueField]);

        const data_ready = pie(data);

        // ✅ Arc generator
        const arc = d3.arc()
            .innerRadius(0) // Set `radius / 3` for donut chart
            .outerRadius(radius);

        const arcLabel = d3.arc()
            .innerRadius(radius * 0.6)
            .outerRadius(radius * 0.9);

        // ✅ Tooltip
        const tooltip = d3.select(container)
            .append("div")
            .style("position", "absolute")
            .style("background", "rgba(0, 0, 0, 0.8)")
            .style("color", "#fff")
            .style("border-radius", "6px")
            .style("padding", "8px")
            .style("font-size", "12px")
            .style("box-shadow", "0px 4px 8px rgba(0,0,0,0.1)")
            .style("display", "none");

        // ✅ Draw Pie Slices
        svg.selectAll("path")
            .data(data_ready)
            .enter()
            .append("path")
            .attr("d", arc)
            .attr("fill", d => color(d.data[labelField]))
            .attr("stroke", "#fff")
            .style("stroke-width", "2px")
            .style("opacity", 0.85)
            .transition()
            .duration(800)
            .attrTween("d", function (d) {
                const i = d3.interpolate({ startAngle: 0, endAngle: 0 }, d);
                return function (t) {
                    return arc(i(t));
                };
            });

        // ✅ Add Interactivity
        svg.selectAll("path")
            .on("mouseover", function (event, d) {
                d3.select(this).transition().duration(200).style("opacity", 1);
                tooltip.style("display", "block")
                    .html(`<strong>Region:</strong> ${d.data[labelField]}<br><strong>No. of Orders:</strong> ${d.data[valueField]}`)
                    .style("left", `${event.pageX + 10}px`)
                    .style("top", `${event.pageY - 20}px`);
            })
            .on("mousemove", function (event) {
                tooltip.style("left", `${event.pageX + 10}px`)
                    .style("top", `${event.pageY - 20}px`);
            })
            .on("mouseout", function () {
                d3.select(this).transition().duration(200).style("opacity", 0.85);
                tooltip.style("display", "none");
            });

        // ✅ Add Labels Outside Slices
        svg.selectAll("text")
            .data(data_ready)
            .enter()
            .append("text")
            .attr("transform", d => `translate(${arcLabel.centroid(d)})`)
            .attr("text-anchor", "middle")
            .attr("font-size", "12px")
            .attr("fill", "#333")
            .style("font-weight", "bold")
            .text(d => `${d.data[labelField]} (${d.data[valueField]})`);
    }
    // Re-render the chart whenever data updates
    $: drawPieChart();
</script>

<div bind:this={container}></div>
