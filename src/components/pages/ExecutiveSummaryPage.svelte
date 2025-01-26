<script>
    import { onMount } from 'svelte';
    import * as d3 from 'd3';
    import DataLoader from '../Dataloader.svelte';

    let data = [];
    let topProduct = { name: 'N/A', sales: 0 };
    let topRegion = { name: 'N/A', profit: 0 };
    let bestSegment = { name: 'N/A', profit: 0 };
    let lowestProfitProduct = { name: 'N/A', profit: 0 };
    let lowestSalesRegion = { name: 'N/A', sales: 0 };
    let lowestContributionSegment = { name: 'N/A', profit: 0 };
    let largestDiscount = { value: 0, product: 'N/A' };
    let highestQuantitySold = { quantity: 0, product: 'N/A' };
    let mostFrequentShippingMode = 'N/A';
    let repeatCustomerPercentage = 0;
    let avgOrderFulfillmentTime = 0;

    function calculateSummary(loadedData) {
        if (!loadedData || loadedData.length === 0) {
            console.error('No data loaded!');
            return;
        }

        const salesByProduct = d3.rollups(
            loadedData,
            (v) => d3.sum(v, (d) => d.Sales || 0),
            (d) => d['Product Name'] || 'Unknown Product'
        );
        const profitByRegion = d3.rollups(
            loadedData,
            (v) => d3.sum(v, (d) => d.Profit || 0),
            (d) => d.Region || 'Unknown Region'
        );
        const profitBySegment = d3.rollups(
            loadedData,
            (v) => d3.sum(v, (d) => d.Profit || 0),
            (d) => d.Segment || 'Unknown Segment'
        );

        salesByProduct.sort((a, b) => b[1] - a[1]);
        profitByRegion.sort((a, b) => b[1] - a[1]);
        profitBySegment.sort((a, b) => b[1] - a[1]);

        topProduct = { name: salesByProduct[0]?.[0] || 'N/A', sales: salesByProduct[0]?.[1] || 0 };
        topRegion = { name: profitByRegion[0]?.[0] || 'N/A', profit: profitByRegion[0]?.[1] || 0 };
        bestSegment = { name: profitBySegment[0]?.[0] || 'N/A', profit: profitBySegment[0]?.[1] || 0 };

        const lowestProfit = d3.min(loadedData, (d) => d.Profit);
        const lowestProfitProductData = loadedData.find((d) => d.Profit === lowestProfit);

        lowestProfitProduct = lowestProfitProductData
            ? { name: lowestProfitProductData['Product Name'], profit: lowestProfitProductData.Profit }
            : { name: 'N/A', profit: 0 };

        const lowestSales = d3.min(profitByRegion, (d) => d[1]);
        lowestSalesRegion = { name: profitByRegion.find((d) => d[1] === lowestSales)?.[0] || 'N/A', sales: lowestSales || 0 };

        const lowestContribution = d3.min(profitBySegment, (d) => d[1]);
        lowestContributionSegment = { name: profitBySegment.find((d) => d[1] === lowestContribution)?.[0] || 'N/A', profit: lowestContribution || 0 };

        const largestDiscountItem = loadedData.reduce((max, d) => (d.Discount > max.Discount ? d : max), {
            Discount: 0,
            'Product Name': 'N/A',
        });
        largestDiscount = { value: largestDiscountItem.Discount * 100, product: largestDiscountItem['Product Name'] };

        const highestQuantityItem = loadedData.reduce((max, d) => (d.Quantity > max.Quantity ? d : max), {
            Quantity: 0,
            'Product Name': 'N/A',
        });
        highestQuantitySold = { quantity: highestQuantityItem.Quantity, product: highestQuantityItem['Product Name'] };

        // Calculate most frequent shipping mode
        const shippingModeCounts = d3.rollups(
            loadedData,
            (v) => v.length,
            (d) => d['Ship Mode']
        );
        shippingModeCounts.sort((a, b) => b[1] - a[1]);
        mostFrequentShippingMode = shippingModeCounts[0]?.[0] || 'N/A';

        // Calculate repeat customer percentage
        const uniqueCustomerCounts = new Set(loadedData.map((d) => d['Customer ID']));
        repeatCustomerPercentage = (1 - uniqueCustomerCounts.size / loadedData.length) * 100;

        // Calculate average order fulfillment time
        const fulfillmentTimes = loadedData.map((d) => {
            const orderDate = new Date(d['Order Date']);
            const shipDate = new Date(d['Ship Date']);
            return (shipDate - orderDate) / (1000 * 60 * 60 * 24); // Convert milliseconds to days
        });
        avgOrderFulfillmentTime = d3.mean(fulfillmentTimes) || 0;
    }
</script>

<DataLoader {data} onDataLoaded={calculateSummary} />

<!-- Summary Section -->
<section style="padding: 1rem; background: #f9f9f9; border-radius: 8px; box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);">
    <h2 style="text-align: center; margin-bottom: 1rem; font-size: 1.5rem; color: #333;">📊 Executive Summary</h2>

    <!-- Key Metrics -->
    <div style="display: flex; flex-wrap: wrap; gap: 1rem; justify-content: space-between;">
        <div style="flex: 1; background: white; padding: 1rem; border-radius: 8px; box-shadow: 0 2px 6px rgba(0, 0, 0, 0.1);">
            <h3>🔝 Top Product</h3>
            <p>{topProduct.name} (${(topProduct.sales || 0).toFixed(2)})</p>
        </div>
        <div style="flex: 1; background: white; padding: 1rem; border-radius: 8px; box-shadow: 0 2px 6px rgba(0, 0, 0, 0.1);">
            <h3>🌍 Top Region</h3>
            <p>{topRegion.name} (${(topRegion.profit || 0).toFixed(2)})</p>
        </div>
        <div style="flex: 1; background: white; padding: 1rem; border-radius: 8px; box-shadow: 0 2px 6px rgba(0, 0, 0, 0.1);">
            <h3>👥 Best Segment</h3>
            <p>{bestSegment.name} (${(bestSegment.profit || 0).toFixed(2)})</p>
        </div>
        <div style="flex: 1; background: white; padding: 1rem; border-radius: 8px; box-shadow: 0 2px 6px rgba(0, 0, 0, 0.1);">
            <h3>📦 Most Frequent Shipping Mode</h3>
            <p>{mostFrequentShippingMode}</p>
        </div>
    </div>

    <!-- Additional Metrics -->
    <div style="display: flex; flex-wrap: wrap; gap: 1rem; margin-top: 1rem;">
        <div style="flex: 1; background: white; padding: 1rem; border-radius: 8px; box-shadow: 0 2px 6px rgba(0, 0, 0, 0.1);">
            <h3>🔁 Repeat Customer Percentage</h3>
            <p>{repeatCustomerPercentage.toFixed(2)}%</p>
        </div>
        <div style="flex: 1; background: white; padding: 1rem; border-radius: 8px; box-shadow: 0 2px 6px rgba(0, 0, 0, 0.1);">
            <h3>⏳ Avg. Order Fulfillment Time</h3>
            <p>{avgOrderFulfillmentTime.toFixed(2)} days</p>
        </div>
        <div style="flex: 1; background: white; padding: 1rem; border-radius: 8px; box-shadow: 0 2px 6px rgba(0, 0, 0, 0.1);">
            <h3>📦 Highest Quantity Sold</h3>
            <p>{highestQuantitySold.quantity} units of {highestQuantitySold.product}</p>
        </div>
    </div>
</section>
