<script>
    import * as d3 from 'd3';
    import { onMount } from 'svelte';

    export let data = [];
    export let xField;
    export let yField;
    
    let container;

    $: if (data && data.length > 0 && container) {
        console.log("📊 Shipping Mode Usage Data:", data);
        drawBarChart();
    }

    function drawBarChart() {
        // ✅ Chart Dimensions
        const margin = { top: 30, right: 30, bottom: 60, left: 60 };
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

        // ✅ Scales
        const xScale = d3.scaleBand()
            .domain(data.map(d => d[xField]))
            .range([0, width])
            .padding(0.3);

        const yScale = d3.scaleLinear()
            .domain([0, d3.max(data, d => d[yField]) || 1])
            .nice()
            .range([height, 0]);

        // ✅ Formal Color Gradient
        const gradient = svg.append("defs")
            .append("linearGradient")
            .attr("id", "barGradient")
            .attr("x1", "0%")
            .attr("y1", "0%")
            .attr("x2", "0%")
            .attr("y2", "100%");

        gradient.append("stop").attr("offset", "0%").attr("stop-color", "#2C3E50");  // Dark Blue
        gradient.append("stop").attr("offset", "100%").attr("stop-color", "#34495E"); // Lighter Blue

        // ✅ Axes
        svg.append("g")
            .attr("transform", `translate(0, ${height})`)
            .call(d3.axisBottom(xScale).tickSize(0))
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
            .attr("y", height + 56)
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
            .text("No. of Orders");

        // ✅ Tooltip
        const tooltip = d3.select(container)
            .append("div")
            .style("position", "absolute")
            .style("background", "#fff")
            .style("border", "1px solid #ccc")
            .style("padding", "8px")
            .style("border-radius", "5px")
            .style("box-shadow", "0px 4px 8px rgba(0,0,0,0.1)")
            .style("display", "none")
            .style("font-size", "14px")
            .style("color", "#333");

        // ✅ Bars
        svg.selectAll(".bar")
            .data(data)
            .enter()
            .append("rect")
            .attr("x", d => xScale(d[xField]))
            .attr("y", d => yScale(d[yField]))
            .attr("width", xScale.bandwidth())
            .attr("height", d => height - yScale(d[yField]))
            .attr("fill", "url(#barGradient)")
            .attr("rx", 6)  // Rounded corners
            .attr("ry", 6)
            .on("mouseover", (event, d) => {
                d3.select(event.target).attr("fill", "#1F618D"); // Darker blue on hover
                tooltip.style("display", "block")
                    .html(`<strong>Shipping Mode:</strong> ${d[xField]}<br><strong>Orders:</strong> ${d[yField]}`)
                    .style("left", `${event.pageX + 10}px`)
                    .style("top", `${event.pageY - 20}px`);
            })
            .on("mouseout", (event, d) => {
                d3.select(event.target).attr("fill", "url(#barGradient)");
                tooltip.style("display", "none");
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
