<script>
    import DataLoader from './Dataloader.svelte';
    import BarChart from './charts/HomePage/BarChart.svelte';
    import LineChart from './charts/HomePage/LineChart.svelte';
    import PieChart from './charts/HomePage/PieChart.svelte';
    import HeatmapChart from './charts/HomePage/HeatmapChart.svelte';

    let data = [];
    let totalSales = 0;
    let totalProfit = 0;
    let profitMargin = 0;
    let totalOrders = 0;
    let averageOrderValue = 0;
    let totalDiscount = 0;
    let dataLoaded = false;

    // Process CSV data and calculate KPIs
    function processData(csvData) {
        data = csvData;

        totalSales = data.reduce((sum, row) => sum + (Number(row.Sales) || 0), 0);
        totalProfit = data.reduce((sum, row) => sum + (Number(row.Profit) || 0), 0);
        totalOrders = data.length;
        averageOrderValue = totalOrders > 0 ? (totalSales / totalOrders).toFixed(2) : 0;
        totalDiscount = data.reduce((sum, row) => sum + (Number(row.Discount) || 0), 0);
        profitMargin = totalSales > 0 ? ((totalProfit / totalSales) * 100).toFixed(2) : "0.00";

        dataLoaded = true; // Trigger rendering
    }
</script>

<style>
    /* KPI Section */
    .kpi-section {
        display: grid;
        grid-template-columns: repeat(auto-fit, minmax(150px, 1fr));
        gap: 1rem;
        padding: 1rem;
        background: #f9f9f9;
        border-radius: 8px;
        box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
    }

    .kpi-card {
        text-align: center;
        padding: 1rem;
        background: white;
        border-radius: 8px;
        box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
    }

    /* .chart-section {
        display: grid;
        grid-template-columns: repeat(2, 1fr);
        gap: 1rem;
        padding: 1rem;
        background: #f9f9f9;
        border-radius: 8px;
        box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
    } */
    .chart-section {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(550px, 1fr));
    /* grid-template-columns: repeat(2, 1fr); */
    gap: 1rem;
    padding: 1rem;
    }

    .chart-container:nth-child(3) {
        grid-column: 1 / -1; /* Full width for third chart */
    }

    .chart-container {
    padding: 1rem;
    background: white;
    border-radius: 8px;
    box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
    min-width: 0; /* Fix grid overflow */
    height: auto; /* Remove fixed height */
}

    /* Mobile responsiveness */
    @media (max-width: 768px) {
    .chart-section {
        grid-template-columns: 1fr;
    }
    
    .chart-container {
        width: 100%;
        margin: 0 auto;
    }
}
</style>

    <DataLoader {data} onDataLoaded={processData} />

        <!-- KPI Section -->
        <section class="kpi-section">
            <div class="kpi-card">
                <h2>💰 Total Sales</h2>
                <p class="kpi-value">${totalSales.toFixed(2)}</p>
                <small class="kpi-description">Total revenue generated from all transactions</small>
            </div>
            <div class="kpi-card">
                <h2>📈 Total Profit</h2>
                <p class="kpi-value">${totalProfit.toFixed(2)}</p>
                <small class="kpi-description">Net earnings after deducting costs</small>
            </div>
            <div class="kpi-card">
                <h2>🎯 Profit Margin</h2>
                <p class="kpi-value">{profitMargin}%</p>
                <small class="kpi-description">Percentage of profit from total revenue</small>
            </div>
            <div class="kpi-card">
                <h2>📦 Total Orders</h2>
                <p class="kpi-value">{totalOrders}</p>
                <small class="kpi-description">Number of transactions recorded</small>
            </div>
            <div class="kpi-card">
                <h2>🛒 Avg. Order Value</h2>
                <p class="kpi-value">${averageOrderValue}</p>
                <small class="kpi-description">Revenue per transaction on average</small>
            </div>
            <div class="kpi-card">
                <h2>💸 Total Discounts</h2>
                <p class="kpi-value">${totalDiscount.toFixed(2)}</p>
                <small class="kpi-description">Total discount amount applied to sales</small>
            </div>
        </section>

    <!-- Charts Section -->
    {#if dataLoaded}
    <section class="chart-section">
        <div class="chart-container">
            <h3 style="text-align: center;">💹 Category Performance: Sales Comparison</h3>
            <!-- <p class="chart-description">Top performing product categories</p> -->
            <BarChart {data} />
        </div>
        <div class="chart-container">
            <h3 style="text-align: center;">📍 Profit Distribution Across Regions</h3>
            <!-- <p class="chart-description">Geographic earnings contribution</p> -->
            <PieChart {data} />
        </div>
        <div class="chart-container">
            <h3 style="text-align: center;">🚀 Profit Trends Over Time</h3>
            <!-- <p class="chart-description">Monthly performance trajectory</p> -->
            <LineChart {data} />
        </div>
        <!-- <div class="chart-container">
            <h3>🔥 Discount Effectiveness Map</h3>
            <p class="chart-description">Price reduction influence analysis</p>
            <HeatmapChart {data} />
        </div> -->
    </section>
    {:else}
        <p style="text-align: center; padding: 2rem;">Loading data and charts...</p>
    {/if}
