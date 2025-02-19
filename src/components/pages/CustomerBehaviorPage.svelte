<script>
    import { onMount } from 'svelte';
    import * as d3 from 'd3';
    import DataLoader from '../Dataloader.svelte';
    import BarChart from '../charts/CustomerBehaviorPage/BarChart.svelte';
    import PieChart from '../charts/CustomerBehaviorPage/PieChart.svelte';
    import LineChart from '../charts/CustomerBehaviorPage/LineChart.svelte';
    import ScatterPlot from '../charts/CustomerBehaviorPage/ScatterPlot.svelte';

    let data = [];
    let repeatCustomerRate = 0;
    let newOrders = 0;
    let repeatOrders = 0;
    let averageOrderValue = 0;
    let topCustomer = { name: 'N/A', sales: 0 };
    let popularCategory = 'N/A';
    let filteredData = [];
    let selectedRegion = 'All Regions';
    let selectedSegment = 'All Segments';
    let startDate, endDate;
    let salesBySegment = [];
    let orderDistributionByRegion = [];
    let monthlyOrderTrends = [];
    let valueDiscountData = [];

    function processCustomerBehavior(loadedData) {
        if (!loadedData || loadedData.length === 0) {
            console.error('No data available!');
            return;
        }

        data = loadedData;
        console.log('Data Loaded:', data);

        // Calculate KPIs
        const customerOrderCounts = d3.rollups(
            data,
            (v) => v.length,
            (d) => d['Customer ID']
        );
        repeatOrders = customerOrderCounts.filter(([_, count]) => count > 1).length;
        newOrders = customerOrderCounts.length - repeatOrders;
        repeatCustomerRate = (repeatOrders / customerOrderCounts.length) * 100 || 0;

        averageOrderValue = d3.mean(data, (d) => d.Sales || 0).toFixed(2);

        const topCustomerData = d3.rollups(
            data,
            (v) => d3.sum(v, (d) => d.Sales || 0),
            (d) => d['Customer Name']
        ).sort((a, b) => b[1] - a[1])[0];
        topCustomer = { name: topCustomerData[0], sales: topCustomerData[1] };

        popularCategory = d3.rollups(
            data,
            (v) => v.length,
            (d) => d.Category
        ).sort((a, b) => b[1] - a[1])[0][0];

        // Filtered Data
        filteredData = data.filter((d) => {
            const regionMatch = selectedRegion === 'All Regions' || d.Region === selectedRegion;
            const segmentMatch = selectedSegment === 'All Segments' || d.Segment === selectedSegment;
            const dateMatch =
                (!startDate || new Date(d['Order Date']) >= new Date(startDate)) &&
                (!endDate || new Date(d['Order Date']) <= new Date(endDate));
            return regionMatch && segmentMatch && dateMatch;
        });

        // Sales by Segment
        salesBySegment = d3.rollups(
            filteredData,
            (v) => d3.sum(v, (d) => d.Sales || 0),
            (d) => d.Segment
        ).map(([key, value]) => ({ segment: key, sales: value }));

        // Process Order Distribution by Region
        orderDistributionByRegion = d3.rollups(
            filteredData,
            (v) => v.length,
            (d) => d.Region
        ).map(([key, value]) => ({ region: key, orders: value }));

        console.log('Processed orderDistributionByRegion:', orderDistributionByRegion);

        const customerOrderCountsMap = new Map(customerOrderCounts);
        
        // Calculate Monthly Trends
        monthlyOrderTrends = d3.rollups(
            filteredData,
            (v) => {
                const ordersByCustomer = v.reduce(
                    (acc, d) => {
                        const customerId = d['Customer ID'];
                        if (!acc[customerId]) {
                            acc[customerId] = 0;
                        }
                        acc[customerId] += 1;
                        return acc;
                    },
                    {}
                );

                const newOrderCount = Object.keys(ordersByCustomer).filter(
                    (id) => customerOrderCountsMap.get(id) === 1
                ).length;

                const repeatOrderCount = Object.keys(ordersByCustomer).filter(
                    (id) => customerOrderCountsMap.get(id) > 1
                ).length;

                return { newOrders: newOrderCount, repeatOrders: repeatOrderCount };
            },
            (d) => d3.timeFormat('%Y-%m')(new Date(d['Order Date']))).map(([key, value]) => ({ month: key, ...value }));

        console.log('Updated Monthly Order Trends:', monthlyOrderTrends);

        // Order Value vs. Discount
        valueDiscountData = filteredData.map((d) => ({
            value: d.Sales || 0,
            discount: (d.Discount || 0) * 100,
        }));

        console.log('Processed Data:', {
            repeatCustomerRate,
            newOrders,
            repeatOrders,
            averageOrderValue,
            topCustomer,
            popularCategory,
            salesBySegment,
            orderDistributionByRegion,
            monthlyOrderTrends,
            valueDiscountData,
        });
    }
    console.log('SalesBySegment:', salesBySegment);
</script>

<DataLoader {data} onDataLoaded={processCustomerBehavior} />

<!-- Header Section -->
<section style="text-align: center; padding: 0.5rem; background: #f9f9f9; border-radius: 8px; box-shadow: 0px 4px 8px rgba(0,0,0,0.1);">
    <h1 style="font-size: 1.5rem; margin-bottom: 0.5rem;">🔍 Understanding Customer Engagement & Retention</h1>
</section>

<!-- Main Layout -->
<div class="main-container">
    <!-- Sidebar Section (Filters & KPIs) -->
    <aside class="sidebar">
        <!-- Filters -->
        <section class="filter-section">
            <h2>Filters</h2>
            <div class="filter-group">
                <label for="region-filter">Region:</label>
                <select id="region-filter" bind:value={selectedRegion} on:change={() => processCustomerBehavior(data)}>
                    <option value="All Regions">All Regions</option>
                    {#each [...new Set(data.map((d) => d.Region))] as region}
                        <option value={region}>{region}</option>
                    {/each}
                </select>
            </div>
            <div class="filter-group">
                <label for="segment-filter">Segment:</label>
                <select id="segment-filter" bind:value={selectedSegment} on:change={() => processCustomerBehavior(data)}>
                    <option value="All Segments">All Segments</option>
                    {#each [...new Set(data.map((d) => d.Segment))] as segment}
                        <option value={segment}>{segment}</option>
                    {/each}
                </select>
            </div>
            <div class="filter-group">
                <label for="date-range">Date Range:</label>
                <input type="date" bind:value={startDate} on:change={() => processCustomerBehavior(data)} />
                <input type="date" bind:value={endDate} on:change={() => processCustomerBehavior(data)} />
            </div>
        </section>

        <!-- KPI Section -->
<!-- KPI Section -->
<section class="kpi-section">
    <h2>📊 Key Metrics</h2>
    <div class="kpi-card">
        <h3>🔄 Repeat Customer Rate</h3>
        <p>{repeatCustomerRate.toFixed(2)}%</p>
    </div>
    <div class="kpi-card">
        <h3>🆕 vs. 🔄 Orders</h3>
        <p>New: {newOrders} | Repeat: {repeatOrders}</p>
    </div>
    <div class="kpi-card">
        <h3>💵 Average Order Value</h3>
        <p>${averageOrderValue}</p>
    </div>
    <div class="kpi-card">
        <h3>🏆 Top Customer</h3>
        <p>{topCustomer.name} (${topCustomer.sales.toFixed(2)})</p>
    </div>
    <div class="kpi-card">
        <h3>🛒 Popular Category</h3>
        <p>{popularCategory}</p>
    </div>
</section>
    </aside>

    <!-- Charts Section (2x2 Grid) -->
    <main class="chart-section">
        <div class="chart-container">
            <h3>👥📈 Customer Segment Performance</h3>
            <BarChart data={salesBySegment} xField="segment" yField="sales" />
        </div>
        <div class="chart-container">
            <h3>🌍 Regional Order Distribution</h3>
            <PieChart data={orderDistributionByRegion} labelField="region" valueField="orders" />
        </div>
        <div class="chart-container">
            <h3>🆕🔄📈 New vs. Repeat Customer Activity</h3>
            <LineChart data={monthlyOrderTrends} xField="month" yFields={['newOrders', 'repeatOrders']} />
        </div>
        <div class="chart-container">
            <h3>💰📉 Value-Discount Relationship</h3>
            <ScatterPlot data={valueDiscountData} xField="value" yField="discount" />
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
        height: 125vh;
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

    .kpi-card p {
        font-size: 1rem;
        font-weight: normal;
        color: #555;
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

    select {
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

