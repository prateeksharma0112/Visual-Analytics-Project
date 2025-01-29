<script>
    import { onMount } from 'svelte';
    import * as d3 from 'd3';

    export let data = [];
    export let xField; // Example: 'month'
    export let yFields = []; // Example: ['newOrders', 'repeatOrders']

    let container; // This will be bound to the div element

    // Ensure the chart only renders when data is available & container exists
    $: if (data && Array.isArray(data) && data.length > 0 && container) {
        console.log("📊 LineChart Received Data:", data);
        drawLineChart();
    }

    function drawLineChart() {
        if (!container) {
            console.error("🚨 LineChart Error: Container not found. Retrying...");
            return;
        }

        console.log("📈 LineChart Debug Data:", { data, xField, yFields });

        if (!Array.isArray(data) || data.length === 0 || !xField || !yFields.length) {
            d3.select(container).html('<p style="color: red; text-align: center;">No valid data for Line Chart</p>');
            console.error("🚨 LineChart Error: Data is empty or invalid.");
            return;
        }

        // ✅ Parse 'YYYY-MM' format into Date objects safely
        const parseTime = d3.timeParse("%Y-%m");
        const parsedData = data
            .map(d => ({
                ...d,
                date: parseTime(d[xField]) || new Date() // Default to current date if parsing fails
            }))
            .filter(d => d.date instanceof Date && !isNaN(d.date)) // Ensure valid dates
            .sort((a, b) => a.date - b.date); // Ensure chronological order

        console.log("🛠 Parsed Data for Line Chart:", parsedData);

        // ✅ Chart Dimensions
        const margin = { top: 40, right: 50, bottom: 60, left: 60 };
        const width = 500 - margin.left - margin.right;
        const height = 300 - margin.top - margin.bottom;

        // ✅ Clear previous chart
        const svg = d3.select(container)
            .html('')
            .append('svg')
            .attr('width', width + margin.left + margin.right)
            .attr('height', height + margin.top + margin.bottom)
            .append('g')
            .attr('transform', `translate(${margin.left}, ${margin.top})`);

        // ✅ X-Axis (Time Scale)
        const xScale = d3.scaleTime()
            .domain(d3.extent(parsedData, d => d.date)) // Min & Max Date
            .range([0, width]);

        // ✅ Y-Axis (Order Counts)
        const yScale = d3.scaleLinear()
            .domain([0, d3.max(parsedData, d => Math.max(...yFields.map(field => d[field] || 0)))])
            .nice()
            .range([height, 0]);

        // ✅ X-Axis Labels
        svg.append('g')
            .attr('transform', `translate(0, ${height})`)
            .call(d3.axisBottom(xScale).tickFormat(d3.timeFormat("%b %Y"))) // Formats as 'Jan 2023'
            .selectAll("text")
            .style("text-anchor", "end")
            .attr("dx", "-.8em")
            .attr("dy", ".15em")
            .attr("transform", "rotate(-30)"); // Rotate for readability

        // ✅ Y-Axis Labels
        svg.append('g').call(d3.axisLeft(yScale));

        // ✅ X & Y Axis Titles
        svg.append("text")
            .attr("x", width / 2)
            .attr("y", height + 50)
            .attr("text-anchor", "middle")
            .style("font-size", "14px")
            .text("Month");

        svg.append("text")
            .attr("x", -height / 2)
            .attr("y", -45)
            .attr("transform", "rotate(-90)")
            .attr("text-anchor", "middle")
            .style("font-size", "14px")
            .text("Order Count");

        // ✅ Grid Lines for Better Readability
        svg.append('g')
            .attr('class', 'grid')
            .call(
                d3.axisLeft(yScale)
                    .tickSize(-width)
                    .tickFormat('')
            )
            .style('stroke-dasharray', '3,3')
            .style('stroke-opacity', 0.5);

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

        // ✅ Plot Lines for each Y-Field
        yFields.forEach((field, index) => {
            const line = d3.line()
                .x(d => xScale(d.date))
                .y(d => yScale(d[field] || 0))
                .curve(d3.curveMonotoneX); // Smooth line

            svg.append('path')
                .datum(parsedData)
                .attr('fill', 'none')
                .attr('stroke', index === 0 ? '#0077cc' : '#f76b1c') // Blue: New Orders, Orange: Repeat Orders
                .attr('stroke-width', 2)
                .attr('d', line);
        });

        // ✅ Add Data Points with Hover Tooltip
        yFields.forEach((field, index) => {
            svg.selectAll(`.dot-${field}`)
                .data(parsedData)
                .enter()
                .append('circle')
                .attr('cx', d => xScale(d.date))
                .attr('cy', d => yScale(d[field] || 0))
                .attr('r', 4)
                .attr('fill', index === 0 ? '#0077cc' : '#f76b1c')
                .on("mouseover", (event, d) => {
                    tooltip.style("display", "block")
                        .html(`<strong>Month:</strong> ${d3.timeFormat("%b %Y")(d.date)}<br>
                               <strong>${field}:</strong> ${d[field]}`)
                        .style("left", `${event.pageX + 10}px`)
                        .style("top", `${event.pageY - 20}px`);
                })
                .on("mousemove", (event) => {
                    tooltip.style("left", `${event.pageX + 10}px`)
                        .style("top", `${event.pageY - 20}px`);
                })
                .on("mouseout", () => {
                    tooltip.style("display", "none");
                });
        });

        console.log("✅ Line Chart Rendered Successfully!");
    }

    onMount(() => {
        if (container) {
            console.log("✅ LineChart Container Mounted Successfully.");
            drawLineChart();
        } else {
            console.warn("🚨 LineChart Container Not Found. Will Retry.");
        }
    });
</script>

<div bind:this={container}></div>
