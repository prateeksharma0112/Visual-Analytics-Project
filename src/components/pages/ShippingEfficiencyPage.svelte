<script>
    import { onMount } from 'svelte';
    import * as d3 from 'd3';
    import DataLoader from '../Dataloader.svelte';
    import BarChart from '../charts/ShippingEfficiencyPage/BatChart.svelte';
    import PieChart from '../charts/ShippingEfficiencyPage/PieChart.svelte';
    import BarChart2 from '../charts/ShippingEfficiencyPage/BarChart2.svelte';
    import Histogram from '../charts/ShippingEfficiencyPage/Histogram.svelte';

    let data = [];
    let filteredData = [];
    let avgFulfillmentTime = 0;
    let fastestShippingMode = 'N/A';
    let slowestShippingMode = 'N/A';
    let delayedShipmentsPercentage = 0;
    let selectedRegion = 'All Regions';
    let selectedShipMode = 'All Shipping Modes';
    let startDate, endDate;
    let fulfillmentTimeByRegion = [];
    let fulfillmentTimeByShipMode = [];
    let shippingModeUsage = [];
    let fulfillmentTimeDistribution = [];
    let delayedOrders = [];

    function processShippingData(loadedData) {
        if (!loadedData || loadedData.length === 0) {
            console.error('No data available!');
            return;
        }

        data = loadedData;

        // ✅ Convert Dates
        data.forEach(d => {
            d['Order Date'] = new Date(d['Order Date']);
            d['Ship Date'] = new Date(d['Ship Date']);
            d['Fulfillment Time'] = (d['Ship Date'] - d['Order Date']) / (1000 * 60 * 60 * 24); // Days
        });

        // ✅ Filter Data for Charts
        filteredData = data.filter(d => {
            const regionMatch = selectedRegion === 'All Regions' || d.Region === selectedRegion;
            const modeMatch = selectedShipMode === 'All Shipping Modes' || d['Ship Mode'] === selectedShipMode;
            const dateMatch =
                (!startDate || new Date(d['Order Date']) >= new Date(startDate)) &&
                (!endDate || new Date(d['Order Date']) <= new Date(endDate));
            return regionMatch && modeMatch && dateMatch;
        });

        // ✅ Calculate Average Fulfillment Time
        avgFulfillmentTime = filteredData.length > 0 
            ? d3.mean(filteredData, d => d['Fulfillment Time']).toFixed(1) 
            : 0;

        // ✅ Fastest & Slowest Shipping Mode
        const shipModeStats = d3.rollups(
            filteredData,
            v => d3.mean(v, d => d['Fulfillment Time']),
            d => d['Ship Mode']
        ).sort((a, b) => a[1] - b[1]);

        fastestShippingMode = shipModeStats.length > 0 ? shipModeStats[0][0] : 'N/A';
        slowestShippingMode = shipModeStats.length > 0 ? shipModeStats[shipModeStats.length - 1][0] : 'N/A';

        // ✅ Shipping Mode Delay Thresholds
        const delayThresholds = {
            'Same Day': 0,
            'First Class': 1,
            'Second Class': 2,
            'Standard Class': 3,
        };

        // ✅ Calculate Delayed Shipments Based on Shipping Mode
        const delayedOrdersCount = filteredData.filter(d => {
            const delayThreshold = delayThresholds[d['Ship Mode']]; // Default threshold is 5 days
            return d['Fulfillment Time'] > delayThreshold;
        }).length;

        // ✅ Calculate Total Orders
        const totalOrders = filteredData.length;

        // ✅ Delayed Shipments Percentage
        delayedShipmentsPercentage = totalOrders > 0 
            ? ((delayedOrdersCount / totalOrders) * 100).toFixed(2) 
            : 0;

        // ✅ Delayed Orders Breakdown (Pie Chart)
        delayedOrders = [
            { label: 'On-Time Orders', value: totalOrders - delayedOrdersCount },
            { label: 'Delayed Orders', value: delayedOrdersCount }
        ];

        // ✅ Average Fulfillment Time by Region (Bar Chart)
        fulfillmentTimeByRegion = d3.rollups(
            filteredData,
            v => d3.mean(v, d => d['Fulfillment Time']),
            d => d.Region
        ).map(([region, time]) => ({ region, fulfillmentTime: time.toFixed(1) }));

        fulfillmentTimeByShipMode = d3.rollups(
            filteredData,
            v => d3.mean(v, d => d['Fulfillment Time']),
            d => d['Ship Mode']
        ).map(([mode, time]) => ({ mode, fulfillmentTime: time.toFixed(1) }));


        // ✅ Shipping Mode Usage (Bar Chart)
        shippingModeUsage = d3.rollups(
            filteredData,
            v => v.length,
            d => d['Ship Mode']
        ).map(([mode, count]) => ({ mode, count }));

        // ✅ Fulfillment Time Distribution (Histogram)
        fulfillmentTimeDistribution = d3.rollups(
            filteredData,
            v => v.length,
            d => Math.round(d['Fulfillment Time'])
        ).map(([days, count]) => ({ days, count }));
    }
</script>

<DataLoader {data} onDataLoaded={processShippingData} />

<!-- 📌 Page Title -->
<section style="text-align: center; padding: 0.5rem; background: #f9f9f9; border-radius: 8px; box-shadow: 0px 4px 8px rgba(0,0,0,0.1);">
    <h1 style="font-size: 1.5rem; margin-bottom: 0.5rem;">🚚 Evaluate Order Fulfillment Speed and Shipping Performance</h1>
</section>

<div class="main-container">
    <!-- 📊 Sidebar (Filters & KPIs) -->
    <aside class="sidebar">
        <section class="filter-section">
            <h2>Filters</h2>
            <div class="filter-group">
                <label for="region-select">Region:</label>
                <select id="region-select" bind:value={selectedRegion} on:change={() => processShippingData(data)}>
                    <option value="All Regions">All Regions</option>
                    {#each [...new Set(data.map(d => d.Region))] as region}
                        <option value={region}>{region}</option>
                    {/each}
                </select>
            </div>
            <div class="filter-group">
                <label for="shipmode-select">Shipping Mode:</label>
                <select id="shipmode-select" bind:value={selectedShipMode} on:change={() => processShippingData(data)}>
                    <option value="All Shipping Modes">All Shipping Modes</option>
                    {#each [...new Set(data.map(d => d['Ship Mode']))] as mode}
                        <option value={mode}>{mode}</option>
                    {/each}
                </select>
            </div>
            <div class="filter-group">
                <label for="start-date">Start Date:</label>
                <input type="date" id="start-date" bind:value={startDate} on:change={() => processShippingData(data)} />
            </div>
            <div class="filter-group">
                <label for="end-date">End Date:</label>
                <input type="date" id="end-date" bind:value={endDate} on:change={() => processShippingData(data)} />
            </div>
        </section>

        <!-- 📈 KPI Metrics -->
        <!-- 📈 KPI Metrics -->
        <section class="kpi-section">
            <h2>📊 Key Metrics</h2>
            <div class="kpi-card"><h2>⏳ Avg. Fulfillment Time</h2><p>{avgFulfillmentTime} days</p></div>
            <div class="kpi-card"><h2>🚀 Fastest Shipping Mode</h2><p>{fastestShippingMode}</p></div>
            <div class="kpi-card"><h2>🐌 Slowest Shipping Mode</h2><p>{slowestShippingMode}</p></div>
            <div class="kpi-card"><h2>⚠️ Delayed Shipments</h2><p>{delayedShipmentsPercentage}%</p></div>
        </section>
    </aside>

    <!-- 📊 Charts Section -->
    <!-- 📊 Charts Section -->
    <main class="chart-section">
        <div class="chart-container">
            <h3>🚚 Shipping Mode Frequency</h3>
            <p class="chart-subtitle">Transport method utilization</p>
            <BarChart data={shippingModeUsage} xField="mode" yField="count" />
        </div>
        <div class="chart-container">
            <h3>🚚 Shipping Performance</h3>
            <p class="chart-subtitle">Order Processing Speed by Shipping Mode</p>
            <BarChart2 data={fulfillmentTimeByShipMode} xField="mode" yField="fulfillmentTime" />
        </div>
        <div class="chart-container">
            <h3>📅📊 Fulfillment Timeline</h3>
            <p class="chart-subtitle">Order processing duration distribution</p>
            <Histogram data={fulfillmentTimeDistribution} xField="days" yField="count" />
        </div>
        <div class="chart-container">
            <h3>🥧⚠️ Delivery Status</h3>
            <p class="chart-subtitle">On-time vs delayed orders</p>
            <PieChart data={delayedOrders} labelField="label" valueField="value" />
        </div>
    </main>

</div>

<style>
    .main-container {
        display: grid;
        grid-template-columns: 250px 1fr;
        gap: 1rem;
    }

    /* Sidebar Styles */
    .sidebar {
        background: #f9f9f9;
        padding: 1rem;
        border-radius: 8px;
        box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
        height: 100%;
        overflow-y: auto;
        display: flex;
        flex-direction: column;
        gap: 1.5rem;
    }
        /* KPI Section */
        .kpi-section {
        display: flex;
        flex-direction: column;
        gap: 1rem;
    }

    .kpi-card {
        background: white;
        border-radius: 12px;
        box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
        padding: 1rem;
        text-align: center;
    }

    .kpi-card h2 {
        font-size: 1.2rem;
        font-weight: bold;
        margin-bottom: 0.5rem;
        color: #333;
    }

    .kpi-card p {
        font-size: 1rem;
        font-weight: normal;
        color: #555;
    }
    .chart-subtitle {
        text-align: center;
        color: #666;
        font-size: 0.9rem;
        margin: -0.5rem 0 1.2rem;
        font-weight: 500;
    }
    
    .kpi-card h2 {
        display: flex;
        align-items: center;
        gap: 0.5rem;
    }
    
    .chart-container h3 {
        display: flex;
        align-items: center;
        justify-content: center;
        gap: 0.5rem;
    }

    /* Filters */
    .filter-section {
        display: flex;
        flex-direction: column;
        gap: 1rem;
    }

    .filter-group {
        display: flex;
        flex-direction: column;
        gap: 0.5rem;
    }

    select, input {
        padding: 0.5rem;
        border-radius: 8px;
        border: 1px solid #ccc;
        font-size: 1rem;
    }

    .chart-section {
        display: grid;
        grid-template-columns: repeat(auto-fit, minmax(450px, 1fr));
        gap: 1rem;
    }

    .chart-container {
        padding: 1.5rem;
        background: white;
        border-radius: 12px;
        box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
    }

    .chart-container h3 {
        font-size: 1.25rem;
        text-align: center;
        margin-bottom: 1rem;
        font-weight: 600;
        color: #333;
    }

    /* Responsive Design */
    @media (max-width: 768px) {
        .main-container {
            grid-template-columns: 1fr;
        }

        .sidebar {
            order: 1;
        }
    }
</style>
