<script>
    import { onMount } from 'svelte';
    import * as d3 from 'd3';

    export let data = [];
    export let xField = 'month'; // Default time field (e.g., 'month')
    export let yFields = ['sales', 'profit']; // Y-axis fields (e.g., Sales, Profit)
    export let timeGranularity = 'Monthly'; // Time granularity (Monthly, Quarterly, Yearly)

    let svg;
    let width = 800;
    let height = 400;
    let margin = { top: 20, right: 5, bottom: 50, left: 60 };

    let chartData = [];
    let xScale, yScale, colorScale;

    function processData() {
        const timeFormatter = getTimeFormatter(timeGranularity);
        chartData = d3.rollups(
            data,
            (v) => ({
                sales: d3.sum(v, (d) => parseFloat(d.Sales) || 0),
                profit: d3.sum(v, (d) => parseFloat(d.Profit) || 0),
            }),
            (d) => timeFormatter(new Date(d['Order Date']))
        )
            .map(([key, value]) => ({ [xField]: key, ...value }))
            .sort((a, b) => {
                const dateA = new Date(a[xField]);
                const dateB = new Date(b[xField]);
                return dateA - dateB;
            });
    }

    function getTimeFormatter(granularity) {
        if (granularity === 'Yearly') return d3.timeFormat('%Y');
        if (granularity === 'Quarterly') return (date) => `${date.getFullYear()} Q${Math.ceil((date.getMonth() + 1) / 3)}`;
        return d3.timeFormat('%Y-%m'); // Default: Monthly
    }

    function createScales() {
        xScale = d3
            .scaleBand()
            .domain(chartData.map((d) => d[xField]))
            .range([margin.left, width - margin.right])
            .padding(0.1);

        yScale = d3
            .scaleLinear()
            .domain([0, d3.max(chartData, (d) => Math.max(d.sales, d.profit))])
            .range([height - margin.bottom, margin.top])
            .nice();

        colorScale = d3.scaleOrdinal().domain(yFields).range(['#4facfe', '#f76b1c']);
    }

    function drawLineChart() {
        const svgElement = d3.select(svg);

        svgElement.selectAll('*').remove();

        // Create lines for each yField
        yFields.forEach((field) => {
            const line = d3
                .line()
                .x((d) => xScale(d[xField]) + xScale.bandwidth() / 2)
                .y((d) => yScale(d[field]))
                .curve(d3.curveMonotoneX);

            svgElement
                .append('path')
                .datum(chartData)
                .attr('d', line)
                .attr('fill', 'none')
                .attr('stroke', colorScale(field))
                .attr('stroke-width', 2);
        });

        // Add circles for data points
        yFields.forEach((field) => {
            svgElement
                .selectAll(`.dot-${field}`)
                .data(chartData)
                .join('circle')
                .attr('class', `dot-${field}`)
                .attr('cx', (d) => xScale(d[xField]) + xScale.bandwidth() / 2)
                .attr('cy', (d) => yScale(d[field]))
                .attr('r', 4)
                .attr('fill', colorScale(field))
                .on('mouseover', function (event, d) {
                    d3.select(this).attr('r', 6);
                    showTooltip(event, d, field);
                })
                .on('mouseout', function () {
                    d3.select(this).attr('r', 4);
                    hideTooltip();
                });
        });

        // Add X-axis
        svgElement
            .append('g')
            .attr('transform', `translate(0,${height - margin.bottom})`)
            .call(d3.axisBottom(xScale).tickSize(0))
            .selectAll('text')
            .attr('transform', 'rotate(-45)')
            .style('text-anchor', 'end')
            .style('font-size', '12px');

        // Add Y-axis with gridlines
        const yAxis = d3.axisLeft(yScale).ticks(6);

        svgElement
            .append('g')
            .attr('transform', `translate(${margin.left},0)`)
            .call(yAxis)
            .call((g) => g.select('.domain').remove())
            .call((g) =>
                g
                    .selectAll('.tick line')
                    .clone()
                    .attr('x2', width - margin.right - margin.left)
                    .attr('stroke-opacity', 0.1)
            )
            .selectAll('text')
            .style('font-size', '12px');

        // Add X-axis label
        svgElement
            .append('text')
            .attr('x', width / 2)
            .attr('y', height - margin.bottom + 50)
            .attr('text-anchor', 'middle')
            .style('font-size', '14px')
            .style('font-weight', 'bold')
            .text('Months');

        // Add Y-axis label
        svgElement
            .append('text')
            .attr('transform', 'rotate(-90)')
            .attr('x', -height / 2)
            .attr('y', margin.left - 50)
            .attr('text-anchor', 'middle')
            .style('font-size', '14px')
            .style('font-weight', 'bold')
            .text('Amount ($)');
    }

    function drawLegend() {
        const legend = d3.select(svg)
            .append('g')
            .attr('transform', `translate(${width - margin.right - 150}, ${margin.top})`);

        yFields.forEach((field, index) => {
            const legendGroup = legend.append('g')
                .attr('transform', `translate(0, ${index * 20})`);

            legendGroup.append('rect')
                .attr('x', 0)
                .attr('y', 0)
                .attr('width', 15)
                .attr('height', 15)
                .attr('fill', colorScale(field));

            legendGroup.append('text')
                .attr('x', 20)
                .attr('y', 12)
                .text(field.charAt(0).toUpperCase() + field.slice(1))
                .style('font-size', '12px');
        });
    }

    let tooltip;

    function createTooltip() {
        tooltip = d3.select('body')
            .append('div')
            .style('position', 'absolute')
            .style('background', '#fff')
            .style('border', '1px solid #ccc')
            .style('border-radius', '4px')
            .style('padding', '8px')
            .style('box-shadow', '0 4px 8px rgba(0,0,0,0.1)')
            .style('visibility', 'hidden');
    }

    function showTooltip(event, d, field) {
        tooltip
            .style('visibility', 'visible')
            .style('top', `${event.pageY + 10}px`)
            .style('left', `${event.pageX + 10}px`)
            .html(`
                <strong>${xField}:</strong> ${d[xField]}<br>
                <strong>${field.charAt(0).toUpperCase() + field.slice(1)}:</strong> $${d[field].toFixed(2)}
            `);
    }

    function hideTooltip() {
        tooltip.style('visibility', 'hidden');
    }

    onMount(() => {
        processData();
        createScales();
        createTooltip();
        drawLineChart();
        drawLegend();
    });

    $: if (data.length) {
        processData();
        createScales();
        drawLineChart();
        drawLegend();
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
