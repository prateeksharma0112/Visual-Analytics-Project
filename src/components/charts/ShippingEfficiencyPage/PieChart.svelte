<script>
    import { onMount } from "svelte";
    import * as d3 from "d3";

    // Input data for the Pie Chart
    export let data = []; // Example: [{ label: "On-Time Orders", value: 500 }, { label: "Delayed Orders", value: 200 }]
    export let labelField = "label"; // Field name for labels (e.g., "On-Time Orders", "Delayed Orders")
    export let valueField = "value"; // Field name for numeric values (e.g., counts)

    let container;

    // Explanation of thresholds:
    // - Orders are considered "Delayed Orders" if their fulfillment time exceeds a threshold (e.g., 5 days).
    // - Orders within the threshold are categorized as "On-Time Orders."

    // Draw the chart whenever the data or container is available
    $: if (data && container) {
        console.log("📊 Delayed Shipments Breakdown Data:", data);
        drawPieChart();
    }

    function drawPieChart() {
        if (!container) {
            console.error("🚨 PieChart Error: Container not found.");
            return;
        }

        console.log("📈 Debug Pie Chart Data:", data);

        // ✅ Chart Dimensions
        const width = 400;
        const height = 400;
        const radius = Math.min(width, height) / 2; // Calculate radius for the Pie Chart

        // ✅ Clear any existing SVG content
        const svg = d3.select(container)
            .html("") // Remove any previously rendered charts
            .append("svg")
            .attr("width", width)
            .attr("height", height)
            .append("g")
            .attr("transform", `translate(${width / 2}, ${height / 2})`); // Center the chart

        // ✅ Color Scale (using formal and visually appealing colors)
        const color = d3.scaleOrdinal()
            .domain(data.map(d => d[labelField])) // Use the label field as domain
            .range(["#5DADE2", "#76D7C4"]); // Light blue and teal for professional appearance

        // ✅ Pie and Arc Generators
        const pie = d3.pie()
            .value(d => d[valueField]) // Use the value field for pie slices
            .sort(null); // Prevent sorting to maintain data order

        const arc = d3.arc()
            .innerRadius(0) // Set inner radius to 0 for a full pie chart
            .outerRadius(radius); // Set outer radius to the calculated value

        // ✅ Tooltip for displaying additional information
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
            .style("opacity", 0); // Start hidden

        // ✅ Draw Pie Slices
        svg.selectAll("path")
            .data(pie(data)) // Bind data to pie slices
            .enter()
            .append("path")
            .attr("d", arc) // Create arc paths
            .attr("fill", d => color(d.data[labelField])) // Assign color based on the label
            .on("mouseover", (event, d) => {
                d3.select(event.target).attr("opacity", 0.8); // Highlight the hovered slice
                tooltip.transition().duration(200).style("opacity", 1); // Show tooltip
                tooltip.html(`
                    <strong>${d.data[labelField]}</strong><br>
                    Orders: ${d.data[valueField]}<br>
                    Share: ${Math.round((d.data[valueField] / d3.sum(data, d => d[valueField])) * 100)}%
                `)
                .style("left", `${event.pageX + 10}px`)
                .style("top", `${event.pageY - 20}px`);
            })
            .on("mousemove", event => {
                tooltip.style("left", `${event.pageX + 10}px`).style("top", `${event.pageY - 20}px`);
            })
            .on("mouseout", event => {
                d3.select(event.target).attr("opacity", 1); // Reset opacity
                tooltip.transition().duration(200).style("opacity", 0); // Hide tooltip
            });

        // ✅ Add Percentage Labels to the Slices
        const labelArc = d3.arc()
            .innerRadius(radius - 40) // Position labels inside the pie
            .outerRadius(radius - 40);

        svg.selectAll("text")
            .data(pie(data)) // Bind data to labels
            .enter()
            .append("text")
            .attr("transform", d => `translate(${labelArc.centroid(d)})`) // Position labels
            .attr("text-anchor", "middle")
            .style("font-size", "12px")
            .style("fill", "#333")
            .text(d => `${Math.round((d.data[valueField] / d3.sum(data, d => d[valueField])) * 100)}%`);
    }
</script>

<div bind:this={container}></div>

