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

    /* Charts Section */
    .chart-section {
        display: grid;
        grid-template-columns: repeat(auto-fit, minmax(450px, 1fr));
        gap: 1rem;
        padding: 1rem;
        background: #f9f9f9;
        border-radius: 8px;
        box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
    }

    .chart-container {
        padding: 1rem;
        background: white;
        border-radius: 8px;
        box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
    }

    /* Media Queries for Smaller Screens */
    @media (max-width: 768px) {
        .kpi-section {
            grid-template-columns: 1fr; /* Stack KPIs vertically */
        }

        .chart-section {
            grid-template-columns: 1fr; /* Stack charts vertically */
        }
    }
</style>

    <DataLoader {data} onDataLoaded={processData} />

    <!-- KPI Section -->
    <section class="kpi-section">
        <div class="kpi-card">
            <h2>Total Sales</h2>
            <p>${totalSales.toFixed(2)}</p>
            <small>Total revenue generated from all transactions</small>
        </div>
        <div class="kpi-card">
            <h2>Total Profit</h2>
            <p>${totalProfit.toFixed(2)}</p>
            <small>Net earnings after deducting costs</small>
        </div>
        <div class="kpi-card">
            <h2>Profit Margin</h2>
            <p>{profitMargin}%</p>
            <small>Percentage of profit from total revenue</small>
        </div>
        <div class="kpi-card">
            <h2>Total Orders</h2>
            <p>{totalOrders}</p>
            <small>Number of transactions recorded</small>
        </div>
        <div class="kpi-card">
            <h2>Avg. Order Value</h2>
            <p>${averageOrderValue}</p>
            <small>Revenue per transaction on average</small>
        </div>
        <div class="kpi-card">
            <h2>Total Discounts</h2>
            <p>${totalDiscount.toFixed(2)}</p>
            <small>Total discount amount applied to sales</small>
        </div>
    </section>

    <!-- Charts Section -->
    {#if dataLoaded}
        <section class="chart-section">
            <div class="chart-container">
                <h3>Sales (Category Wise)</h3>
                <BarChart {data} />
            </div>
            <div class="chart-container">
                <h3>Profit Distribution (Region Wise)</h3>
                <PieChart {data} />
            </div>
            <div class="chart-container">
                <h3>Profit Trends Over Time</h3>
                <LineChart {data} />
            </div>
            <div class="chart-container">
                <h3>Discount Impact on Sales</h3>
                <HeatmapChart {data} />
            </div>
        </section>
    {:else}
        <p style="text-align: center; padding: 2rem;">Loading data and charts...</p>
    {/if}
