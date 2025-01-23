<script>
    import Navbar from './Navbar.svelte';
    import DataLoader from './Dataloader.svelte';
    import BarChart from './charts/BarChart.svelte';
    import LineChart from './charts/LineChart.svelte';
    import PieChart from './charts/PieChart.svelte';
    import HeatmapChart from './charts/HeatmapChart.svelte';

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

<!-- Navbar -->
<Navbar title="Superstore Sales Dashboard" />
<DataLoader {data} onDataLoaded={processData} />

<!-- KPI Section -->
<section style="
    display: flex; 
    flex-wrap: wrap; 
    justify-content: space-between; 
    align-items: center; 
    padding: 1rem; 
    background: #f9f9f9; 
    box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1); 
    margin-bottom: 1rem; 
    border-radius: 8px;
">
    <div style="text-align: center; flex: 1; margin: 0.5rem;">
        <h2 style="font-size: 0.9rem; color: #333;">Total Sales</h2>
        <p style="font-size: 1.2rem; font-weight: bold;">${totalSales.toFixed(2)}</p>
        <small style="font-size: 0.8rem; color: #666;">Total revenue generated from all transactions</small>
    </div>
    <div style="text-align: center; flex: 1; margin: 0.5rem;">
        <h2 style="font-size: 0.9rem; color: #333;">Total Profit</h2>
        <p style="font-size: 1.2rem; font-weight: bold;">${totalProfit.toFixed(2)}</p>
        <small style="font-size: 0.8rem; color: #666;">Net earnings after deducting costs</small>
    </div>
    <div style="text-align: center; flex: 1; margin: 0.5rem;">
        <h2 style="font-size: 0.9rem; color: #333;">Profit Margin</h2>
        <p style="font-size: 1.2rem; font-weight: bold;">{profitMargin}%</p>
        <small style="font-size: 0.8rem; color: #666;">Percentage of profit from total revenue</small>
    </div>
    <div style="text-align: center; flex: 1; margin: 0.5rem;">
        <h2 style="font-size: 0.9rem; color: #333;">Total Orders</h2>
        <p style="font-size: 1.2rem; font-weight: bold;">{totalOrders}</p>
        <small style="font-size: 0.8rem; color: #666;">Number of transactions recorded</small>
    </div>
    <div style="text-align: center; flex: 1; margin: 0.5rem;">
        <h2 style="font-size: 0.9rem; color: #333;">Avg. Order Value</h2>
        <p style="font-size: 1.2rem; font-weight: bold;">${averageOrderValue}</p>
        <small style="font-size: 0.8rem; color: #666;">Revenue per transaction on average</small>
    </div>
    <div style="text-align: center; flex: 1; margin: 0.5rem;">
        <h2 style="font-size: 0.9rem; color: #333;">Total Discounts</h2>
        <p style="font-size: 1.2rem; font-weight: bold;">${totalDiscount.toFixed(2)}</p>
        <small style="font-size: 0.8rem; color: #666;">Total discount amount applied to sales</small>
    </div>
</section>

<!-- Charts Section -->
{#if dataLoaded}
    <section style="
        display: grid; 
        grid-template-columns: 1fr 1fr; 
        gap: 1rem; 
        padding: 1rem; 
        background: #f9f9f9; 
        border-radius: 8px; 
        box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
    ">
        <div>
            <h3 style="text-align: center; margin-bottom: 1rem;">Sales by Category</h3>
            <BarChart {data} />
        </div>
        <div>
            <h3 style="text-align: center; margin-bottom: 1rem;">Profit Distribution by Region</h3>
            <PieChart {data} />
        </div>
        <div>
            <h3 style="text-align: center; margin-bottom: 1rem;">Profit Trends Over Time</h3>
            <LineChart {data} />
        </div>
        <div>
            <h3 style="text-align: center; margin-bottom: 1rem;">Discount Impact on Sales</h3>
            <HeatmapChart {data} />
        </div>
    </section>
{:else}
    <p style="text-align: center; padding: 2rem;">Loading data and charts...</p>
{/if}

<!-- Footer -->
<footer style="
    text-align: center; 
    padding: 1rem; 
    background: #333; 
    color: white; 
    margin-top: 1rem; 
    border-radius: 8px;
">
    <p>© 2024 Superstore Sales Dashboard</p>
</footer>
