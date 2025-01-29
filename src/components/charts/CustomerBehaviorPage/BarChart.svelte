<script>
    import * as d3 from 'd3';
    import { onMount } from 'svelte';

    export let data = [];
    export let xField;
    export let yField;

    let container;

    $: {
        console.log('BarChart Data:', data);
        console.log('X Field:', xField);
        console.log('Y Field:', yField);
        drawBarChart();
    }

    function drawBarChart() {
        // ✅ Chart Dimensions
        const margin = { top: 30, right: 20, bottom: 60, left: 60 };
        const width = 450 - margin.left - margin.right;
        const height = 350 - margin.top - margin.bottom;

        // ✅ Clear & Create SVG
        const svg = d3
            .select(container)
            .html('')
            .append('svg')
            .attr('width', width + margin.left + margin.right)
            .attr('height', height + margin.top + margin.bottom)
            .append('g')
            .attr('transform', `translate(${margin.left}, ${margin.top})`);

        // ✅ Scales
        const xScale = d3
            .scaleBand()
            .domain(data.map(d => d[xField]))
            .range([0, width])
            .padding(0.2);

        const yScale = d3
            .scaleLinear()
            .domain([0, d3.max(data, d => d[yField]) || 1])
            .nice()
            .range([height, 0]);

        // ✅ Define Formal Bar Color
        const barColor = "#2E86C1"; // Royal Blue
        const barHoverColor = "#1B4F72"; // Darker Blue for hover

        // ✅ Axes
        svg.append("g")
            .attr("transform", `translate(0, ${height})`)
            .call(d3.axisBottom(xScale).tickSize(0))
            .selectAll("text")
            .attr("transform", "rotate(-30)")
            .style("text-anchor", "end");

        svg.append("g").call(d3.axisLeft(yScale));

        // ✅ X & Y Labels
        svg.append("text")
            .attr("x", width / 2)
            .attr("y", height + 50)
            .attr("text-anchor", "middle")
            .style("font-size", "14px")
            .text("Cutsomer Segment");

        svg.append("text")
            .attr("x", -height / 2)
            .attr("y", -50)
            .attr("transform", "rotate(-90)")
            .attr("text-anchor", "middle")
            .style("font-size", "14px")
            .text("Sales ($)");

        // ✅ Tooltip
        const tooltip = d3.select(container)
            .append("div")
            .style("position", "absolute")
            .style("background", "#fff")
            .style("border", "1px solid #ccc")
            .style("padding", "6px")
            .style("border-radius", "5px")
            .style("box-shadow", "0px 4px 8px rgba(0,0,0,0.1)")
            .style("display", "none");

        // ✅ Bars
        svg.selectAll(".bar")
            .data(data)
            .enter()
            .append("rect")
            .attr("x", d => xScale(d[xField]))
            .attr("y", d => yScale(d[yField]))
            .attr("width", xScale.bandwidth())
            .attr("height", d => height - yScale(d[yField]))
            .attr("fill", barColor)
            .attr("rx", 6) // Rounded corners
            .attr("ry", 6) // Rounded corners
            .on("mouseover", (event, d) => {
                d3.select(event.target).attr("fill", barHoverColor);
                tooltip.style("display", "block")
                    .html(`<strong>${xField}:</strong> ${d[xField]}<br><strong>${yField}:</strong> ${d[yField].toFixed(2)}`)
                    .style("left", `${event.pageX + 10}px`)
                    .style("top", `${event.pageY - 20}px`);
            })
            .on("mouseout", (event, d) => {
                d3.select(event.target).attr("fill", barColor);
                tooltip.style("display", "none");
            });

        // ✅ Labels on Bars (Rounded Values)
        svg.selectAll(".label")
            .data(data)
            .enter()
            .append("text")
            .attr("x", d => xScale(d[xField]) + xScale.bandwidth() / 2)
            .attr("y", d => yScale(d[yField]) - 5)
            .attr("text-anchor", "middle")
            .style("font-size", "14px")
            .style("fill", "#333")
            .text(d => d[yField].toFixed(2));
    }
    onMount(() => {
        drawBarChart();
    });
</script>

<div bind:this={container}></div>
