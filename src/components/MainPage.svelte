<script>
    import Navbar from './Navbar.svelte';
    import DataLoader from './Dataloader.svelte';
    import BarChart from './charts/BarChart.svelte';
    import LineChart from './charts/LineChart.svelte';
    import HeatmapChart from './charts/HeatmapChart.svelte';
    import PieChart from './charts/PieChart.svelte';

    let data = [];
    let totalSales = 0;
    let totalProfit = 0;
    let profitMargin = 0;
    let dataLoaded = false; // New state to ensure charts load after data is available

    // Processing CSV and Calculating KPIs
    function processData(csvData) {
        data = csvData;
        totalSales = data.reduce((sum, row) => sum + (Number(row.Sales) || 0), 0);
        totalProfit = data.reduce((sum, row) => sum + (Number(row.Profit) || 0), 0);
        profitMargin = totalSales > 0 ? ((totalProfit / totalSales) * 100).toFixed(2) : "0.00";
        dataLoaded = true; // Trigger chart rendering once data is ready
    }
</script>

<!-- Navbar (Already Set) -->
<Navbar title="Superstore Sales Dashboard" />
<DataLoader {data} onDataLoaded={processData} />

<!-- KPI Section -->
<section style="display: flex; justify-content: space-around; padding: 2rem; background: #f5f5f5;">
    <div style="text-align: center;">
        <h2>Total Sales</h2>
        <p style="font-size: 2rem; font-weight: bold;">${totalSales.toFixed(2)}</p>
    </div>
    <div style="text-align: center;">
        <h2>Total Profit</h2>
        <p style="font-size: 2rem; font-weight: bold;">${totalProfit.toFixed(2)}</p>
    </div>
    <div style="text-align: center;">
        <h2>Profit Margin</h2>
        <p style="font-size: 2rem; font-weight: bold;">{profitMargin}%</p>
    </div>
</section>

<!-- Charts Section - Render only when data is available -->
{#if dataLoaded}
    <section style="padding: 2rem; display: flex; justify-content: space-around;">
        <div style="width: 45%;">
            <h3>Sales by Category</h3>
            <BarChart {data} />
        </div>
        <div style="width: 45%;">
            <h3>Profit Trends Over Time</h3>
            <LineChart {data} />
        </div>
    </section>

    <section style="padding: 2rem; display: flex; justify-content: space-around;">
        <div style="width: 45%;">
            <h3>Profit Distribution by Region</h3>
            <PieChart {data} />
        </div>
        <div style="width: 45%;">
            <h3>Sales Distribution by Customer Segment</h3>
        <HeatmapChart {data} />
        </div>
    </section>
{:else}
    <p style="text-align: center; padding: 2rem;">Loading data and charts...</p>
{/if}

<!-- Footer -->
<footer style="text-align: center; padding: 1rem; background: #333; color: white;">
    <p>© 2024 Superstore Sales Dashboard</p>
</footer>
