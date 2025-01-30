<script>
    import * as d3 from 'd3';
    import { onMount } from "svelte";
    import DataLoader from '../Dataloader.svelte';
    import Table from '../charts/ProductPerformancePage/Table.svelte';

    let data = []; // Stores full dataset
    let filteredData = []; // Stores filtered data

    // KPIs
    let totalSales = 0;
    let totalProfit = 0;
    let avgDiscount = 0;
    let bestSellingProduct = { name: "N/A", sales: 0 };
    let highestProfitProduct = { name: "N/A", profit: 0 };
    let selectedCategory = "All Categories";
    let selectedRegion = "All Regions";
    let startDate, endDate;
    let searchQuery = "";

    function processProductData(loadedData) {
        if (!loadedData || loadedData.length === 0) {
            console.error("No product data available!");
            return;
        }

        data = loadedData.map(d => ({
            productId: d["Product ID"],
            productName: d["Product Name"],
            category: d["Category"],
            subCategory: d["Sub-Category"],
            totalSales: +d["Sales"],
            totalQuantity: +d["Quantity"],
            avgDiscount: (+d["Discount"] * 100).toFixed(1),
            totalProfit: +d["Profit"],
            profitMargin: ((+d["Profit"] / +d["Sales"]) * 100).toFixed(1),
            region: d["Region"],
            orderDate: new Date(d["Order Date"]),
        }));

        applyFilters();
    }

    function applyFilters() {
        filteredData = data.filter(d => {
            const categoryMatch = selectedCategory === "All Categories" || d.category === selectedCategory;
            const regionMatch = selectedRegion === "All Regions" || d.region === selectedRegion;
            const dateMatch = (!startDate || d.orderDate >= new Date(startDate)) &&
                              (!endDate || d.orderDate <= new Date(endDate));
            const searchMatch = d.productName.toLowerCase().includes(searchQuery.toLowerCase());

            return categoryMatch && regionMatch && dateMatch && searchMatch;
        });

        calculateKPIs();
    }

    function calculateKPIs() {
        if (filteredData.length === 0) {
            totalSales = totalProfit = avgDiscount = 0;
            bestSellingProduct = { name: "N/A", sales: 0 };
            highestProfitProduct = { name: "N/A", profit: 0 };
            return;
        }

        totalSales = d3.sum(filteredData, d => d.totalSales).toFixed(2);
        totalProfit = d3.sum(filteredData, d => d.totalProfit).toFixed(2);
        avgDiscount = (d3.mean(filteredData, d => d.avgDiscount) || 0).toFixed(1);

        const bestProduct = filteredData.reduce((max, d) => d.totalSales > max.sales ? { name: d.productName, sales: d.totalSales } : max, { name: "N/A", sales: 0 });
        bestSellingProduct = bestProduct;

        const profitableProduct = filteredData.reduce((max, d) => d.totalProfit > max.profit ? { name: d.productName, profit: d.totalProfit } : max, { name: "N/A", profit: 0 });
        highestProfitProduct = profitableProduct;
    }

</script>

<DataLoader {data} onDataLoaded={processProductData} />

<!-- 📌 Page Title -->
<section class="page-title">
    <h1>📦 Product Performance Anlysis</h1>
    <p>Gain deep insights into Product Sales, Profitability, and Discount Trends. 
        Identify Top-Performing Products and Optimize Strategies for Growth.</p>
</section>

<div class="main-container">
    <!-- 📊 Sidebar with KPI Metrics -->
    <aside class="sidebar">
        <section class="kpi-section">
            <h2>📊 Key Metrics</h2>
            <div class="kpi-card"><h2>💰 Total Sales</h2><p>${totalSales}</p></div>
            <div class="kpi-card"><h2>📈 Total Profit</h2><p>${totalProfit}</p></div>
            <div class="kpi-card"><h2>⚡ Avg. Discount</h2><p>{avgDiscount}%</p></div>
            <div class="kpi-card"><h2>🔥 Best-Selling Product</h2><p>{bestSellingProduct.name} (${bestSellingProduct.sales})</p></div>
            <div class="kpi-card"><h2>💎 Most Profitable Product</h2><p>{highestProfitProduct.name} (${highestProfitProduct.profit})</p></div>
        </section>
    </aside>

    <!-- 📌 Product Performance Table Section -->
    <main class="chart-section">
        <!-- 📌 Filters Above Table -->
        <div class="filter-container">
            <input type="text" bind:value={searchQuery} placeholder="🔎 Search Product" on:input={applyFilters} />
            <select bind:value={selectedCategory} on:change={applyFilters}>
                <option value="All Categories">All Categories</option>
                {#each [...new Set(data.map(d => d.category))] as category}
                    <option value={category}>{category}</option>
                {/each}
            </select>
            <select bind:value={selectedRegion} on:change={applyFilters}>
                <option value="All Regions">All Regions</option>
                {#each [...new Set(data.map(d => d.region))] as region}
                    <option value={region}>{region}</option>
                {/each}
            </select>
            <input type="date" bind:value={startDate} on:change={applyFilters} />
            <input type="date" bind:value={endDate} on:change={applyFilters} />
        </div>

        <!-- 📌 Product Performance Table -->
        <div class="chart-container">
            <Table data={filteredData} />
        </div>
    </main>
</div>

<style>
    .page-title {
        text-align: center;
        background: #f9f9f9;
        padding: 1rem;
        border-radius: 8px;
        box-shadow: 0px 4px 8px rgba(0,0,0,0.1);
    }

    .main-container {
        display: grid;
        grid-template-columns: 250px 1fr;
        gap: 1rem;
    }

    /* Sidebar */
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

    /* Filters Above Table */
    .filter-container {
        display: flex;
        justify-content: space-between;
        gap: 1rem;
        padding: 1rem;
        background: white;
        border-radius: 8px;
        box-shadow: 0px 4px 8px rgba(0,0,0,0.1);
        margin-bottom: 1rem;
    }

    .filter-container input, .filter-container select {
        padding: 0.5rem;
        border-radius: 8px;
        border: 1px solid #ccc;
        font-size: 1rem;
        flex: 1;
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

    .chart-section {
        display: flex;
        flex-direction: column;
        gap: 1rem;
    }

    .chart-container {
        padding: 1.5rem;
        background: white;
        border-radius: 12px;
        box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
    }

    @media (max-width: 768px) {
        .filter-container {
            flex-direction: column;
        }
    }
</style>
