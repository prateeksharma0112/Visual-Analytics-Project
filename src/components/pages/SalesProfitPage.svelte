<script>
    import { onMount } from 'svelte';
    import * as d3 from 'd3';
    import DataLoader from '../Dataloader.svelte';
    import Heatmap from '../charts/SalesAndProfitPage/Heatmap.svelte';
    import BarChart from '../charts/SalesAndProfitPage/BarChart.svelte';
    import LineChart from '../charts/SalesAndProfitPage/LineChart.svelte';
    import Treemap from '../charts/SalesAndProfitPage/Treemap.svelte';

    
    let data = [];
    let filteredData = [];
    let totalSales = 0;
    let totalProfit = 0;
    let profitMargin = 0;
    let selectedRegion = 'All Regions';
    let selectedCategory = 'All Categories';
    let selectedTime = 'Yearly';
    
    // Function to filter data by time
    function filterByTime(orderDate) {
        const date = new Date(orderDate); // Parse the date
        const now = new Date();

        if (selectedTime === 'Yearly') {
            return date.getFullYear() === now.getFullYear();
        } else if (selectedTime === 'Quarterly') {
            const currentQuarter = Math.floor(now.getMonth() / 3) + 1;
            const orderQuarter = Math.floor(date.getMonth() / 3) + 1;
            return date.getFullYear() === now.getFullYear() && currentQuarter === orderQuarter;
        } else if (selectedTime === 'Monthly') {
            return (
                date.getFullYear() === now.getFullYear() && 
                date.getMonth() === now.getMonth()
            );
        }
        return true;
    }

    // Function to calculate metrics and process data
    function processSummary(loadedData) {
        data = loadedData;

        // Calculate KPIs
        totalSales = d3.sum(data, (d) => parseFloat(d.Sales) || 0);
        totalProfit = d3.sum(data, (d) => parseFloat(d.Profit) || 0);
        profitMargin = totalSales ? ((totalProfit / totalSales) * 100).toFixed(2) : 0;

        // Filter data
        filteredData = data.filter((d) => {
            const regionMatch = selectedRegion === 'All Regions' || d.Region === selectedRegion;
            const categoryMatch = selectedCategory === 'All Categories' || d.Category === selectedCategory;
            const timeMatch = filterByTime(d['Order Date']);
            return regionMatch && categoryMatch;
        });
    }

    // Handle data load
function handleDataLoaded(loadedData) {
    // Parse Order Date as a Date object
    loadedData.forEach((d) => {
        d['Order Date'] = new Date(d['Order Date']); // Parse MM/DD/YYYY format
    });
    processSummary(loadedData);
}

    // Trigger updates when filters change
    function updateFilters() {
        processSummary(data);
    }
</script>

<DataLoader onDataLoaded={handleDataLoaded} />

<!-- Header Section -->
<section style="text-align: center; padding: 0.5rem; background: #f9f9f9; border-radius: 8px; box-shadow: 0px 4px 8px rgba(0,0,0,0.1);">
    <h1 style="font-size: 1.5rem; margin-bottom: 0.5rem;">📊 Sales & Profit Analysis</h1>
</section>

<!-- Filter Section -->
<div class="main-container">
    <!-- Sidebar Section -->
    <aside class="sidebar">
        <section class="filter-section">
            <div class="filter-group">
                <label for="region-select">Region:</label>
                <select id="region-select" bind:value={selectedRegion} on:change={updateFilters}>
                    <option value="All Regions">All Regions</option>
                    {#each [...new Set(data.map((d) => d.Region))] as region}
                        <option value={region}>{region}</option>
                    {/each}
                </select>
            </div>
            <div class="filter-group">
                <label for="category-select">Category:</label>
                <select id="category-select" bind:value={selectedCategory} on:change={updateFilters}>
                    <option value="All Categories">All Categories</option>
                    {#each [...new Set(data.map((d) => d.Category))] as category}
                        <option value={category}>{category}</option>
                    {/each}
                </select>
            </div>
        </section>
        <!-- KPI Section -->
        <section class="kpi-section"> 
            <div class="kpi-card">
                <h2>Total Sales</h2>
                <p>${totalSales.toFixed(2)}</p>
            </div>
            <div class="kpi-card">
                <h2>Total Profit</h2>
                <p>${totalProfit.toFixed(2)}</p>
            </div>
            <div class="kpi-card">
                <h2>Profit Margin</h2>
                <p>{profitMargin}%</p>
            </div>
        </section>
    </aside>

    <!-- Content Section -->
    <main class="content">
        <section class="chart-section">
            <div class="chart-container">
                <h3>Sales & Profit Distribution</h3>
                <Heatmap data={filteredData} />
            </div>
            <div class="chart-container">
                <h3>Sales & Profit (Category Wise) </h3>
                <BarChart data={filteredData} xField="Category" yFields={['sales', 'profit']} />
            </div>
            <div class="chart-container">
                <h3>Monthly Trends</h3>
                <LineChart data={filteredData} timeFilter={selectedTime} />
            </div>
            <div class="chart-container">
                <h3>Contribution (Sub-Category Wise)</h3>
                <Treemap data={filteredData} />
            </div>
        </section>
    </main>
</div>

<style>
    .main-container {
        display: grid;
        grid-template-columns: 250px 1fr;
        gap: 1rem;
    }

    /* Sidebar */
    /* .sidebar {
        background: #f9f9f9;
        padding: 1rem;
        border-radius: 8px;
        box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
        height: 100vh;
        overflow-y: auto;
    } */
         /* Sidebar Styles */
    .sidebar {
        background: #f9f9f9;
        padding: 1rem;
        border-radius: 8px;
        box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
        height: 100vh;
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

    select {
        padding: 0.5rem;
        border-radius: 8px;
        border: 1px solid #ccc;
        font-size: 1rem;
    }

    /* Content */
    .content {
        display: flex;
        flex-direction: column;
        gap: 1rem;
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

        .content {
            order: 2;
        }
    }
</style>

