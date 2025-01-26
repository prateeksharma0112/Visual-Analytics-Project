<script>
    import { onMount } from 'svelte';
    import * as d3 from 'd3';

    export let data = []; // Raw data from parent
    let svg;
    let width = 800;
    let height = 400;
    let treemapData = [];
    let tooltip;

    function processTreemapData() {
        if (!data || data.length === 0) {
            console.warn("No data provided for treemap.");
            return;
        }

        const normalizedData = data.map((d) => ({
            Category: d.Category?.trim(),
            SubCategory: d['Sub-Category']?.trim(),
            Sales: parseFloat(d.Sales) || 0,
            Profit: parseFloat(d.Profit) || 0,
            ProfitMargin: ((parseFloat(d.Profit) || 0) / (parseFloat(d.Sales) || 1)) * 100,
        }));

        const groupedData = d3.rollups(
            normalizedData,
            (v) => ({
                Sales: d3.sum(v, (d) => d.Sales),
                Profit: d3.sum(v, (d) => d.Profit),
                ProfitMargin: d3.mean(v, (d) => d.ProfitMargin),
            }),
            (d) => d.Category,
            (d) => d.SubCategory
        );

        const hierarchy = d3
            .hierarchy({
                children: groupedData.map(([key, children]) => ({
                    name: key,
                    children: children.map(([subKey, values]) => ({
                        name: subKey,
                        Sales: values.Sales,
                        Profit: values.Profit,
                        ProfitMargin: values.ProfitMargin,
                    })),
                })),
            })
            .sum((d) => d.Sales)
            .sort((a, b) => b.value - a.value);

        const treemapLayout = d3
            .treemap()
            .size([width, height])
            .paddingOuter(5)
            .paddingInner(2);

        treemapLayout(hierarchy);
        treemapData = hierarchy.leaves();
    }

    function drawTreemap() {
        if (treemapData.length === 0) {
            console.warn("No treemap data to draw.");
            return;
        }

        const svgElement = d3.select(svg);
        svgElement.selectAll("*").remove();

        svgElement
            .selectAll("rect")
            .data(treemapData)
            .join("rect")
            .attr("x", (d) => d.x0)
            .attr("y", (d) => d.y0)
            .attr("width", (d) => Math.max(0, d.x1 - d.x0))
            .attr("height", (d) => Math.max(0, d.y1 - d.y0))
            .attr("fill", (d) => d3.interpolateBlues((d.data.ProfitMargin + 100) / 200))
            .attr("stroke", "#fff")
            .attr("stroke-width", 1)
            .on("mouseover", function (event, d) {
                d3.select(this).attr("opacity", 0.8);
                showTooltip(event, d);
            })
            .on("mouseout", function () {
                d3.select(this).attr("opacity", 1);
                hideTooltip();
            });

        svgElement
            .selectAll("text")
            .data(treemapData)
            .join("text")
            .attr("x", (d) => (d.x0 + d.x1) / 2)
            .attr("y", (d) => (d.y0 + d.y1) / 2)
            .attr("text-anchor", "middle")
            .attr("font-size", (d) => Math.min((d.x1 - d.x0) / 6, (d.y1 - d.y0) / 3, 12))
            .attr("fill", "#fff")
            .text((d) => (d.x1 - d.x0 > 40 && d.y1 - d.y0 > 15 ? d.data.name : ""));
    }

    function createTooltip() {
        tooltip = d3
            .select("body")
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
        tooltip
            .style("visibility", "visible")
            .style("top", `${event.pageY + 10}px`)
            .style("left", `${event.pageX + 10}px`)
            .html(`
                <strong>Category:</strong> ${d.parent.data.name}<br>
                <strong>Sub-Category:</strong> ${d.data.name}<br>
                <strong>Sales:</strong> $${d.data.Sales.toFixed(2)}<br>
                <strong>Profit:</strong> $${d.data.Profit.toFixed(2)}<br>
                <strong>Profit Margin:</strong> ${d.data.ProfitMargin.toFixed(2)}%
            `);
    }

    function hideTooltip() {
        tooltip.style("visibility", "hidden");
    }

    onMount(() => {
        processTreemapData();
        createTooltip();
        drawTreemap();
    });

    $: if (data.length) {
        processTreemapData();
        drawTreemap();
    }
</script>

<svg bind:this={svg} viewBox={`0 0 ${width} ${height}`} preserveAspectRatio="xMidYMid meet"></svg>

<style>
    svg {
        display: block;
        margin: auto;
        max-width: 100%;
        height: auto;
    }
</style>
