<script>
    import { onMount } from "svelte";

    export let data = []; // Processed data from parent component
    let sortedData = [];
    let sortKey = "totalSales"; // Default sorting by Sales
    let sortOrder = "desc"; // Default descending order
    let searchQuery = ""; // Search query input

    // Pagination
    let currentPage = 1;
    let itemsPerPage = 12;
    let totalPages = 1;

    // Sort & filter data when it updates
    $: {
        let filteredData = data
            .filter(d => d.productName.toLowerCase().includes(searchQuery.toLowerCase()))
            .sort((a, b) => {
                let valA = a[sortKey];
                let valB = b[sortKey];
                if (typeof valA === "string") valA = valA.toLowerCase();
                if (typeof valB === "string") valB = valB.toLowerCase();
                return sortOrder === "asc" ? (valA > valB ? 1 : -1) : (valA < valB ? 1 : -1);
            });

        totalPages = Math.ceil(filteredData.length / itemsPerPage);
        sortedData = filteredData.slice((currentPage - 1) * itemsPerPage, currentPage * itemsPerPage);
    }

    function sortTable(column) {
        if (sortKey === column) {
            sortOrder = sortOrder === "asc" ? "desc" : "asc";
        } else {
            sortKey = column;
            sortOrder = "desc"; // Default sorting to descending
        }
    }

    function nextPage() {
        if (currentPage < totalPages) currentPage++;
    }

    function prevPage() {
        if (currentPage > 1) currentPage--;
    }
</script>

<!-- 📌 Table Container -->
<div class="table-container">
    <table>
        <thead>
            <tr>
                <th on:click={() => sortTable("productName")}>📦 Product Name {sortKey === "productName" ? (sortOrder === "asc" ? "↑" : "↓") : ""}</th>
                <th on:click={() => sortTable("category")}>🗂️ Category {sortKey === "category" ? (sortOrder === "asc" ? "↑" : "↓") : ""}</th>
                <th on:click={() => sortTable("subCategory")}>🏷️ Sub-Category {sortKey === "subCategory" ? (sortOrder === "asc" ? "↑" : "↓") : ""}</th>
                <th on:click={() => sortTable("segment")}>👥 Segment {sortKey === "segment" ? (sortOrder === "asc" ? "↑" : "↓") : ""}</th>
                <th on:click={() => sortTable("totalSales")}>💰 Sales ($) {sortKey === "totalSales" ? (sortOrder === "asc" ? "↑" : "↓") : ""}</th>
                <th on:click={() => sortTable("totalQuantity")}>📦 Quantity Sold {sortKey === "totalQuantity" ? (sortOrder === "asc" ? "↑" : "↓") : ""}</th>
                <th on:click={() => sortTable("avgDiscount")}>⚡ Avg. Discount (%) {sortKey === "avgDiscount" ? (sortOrder === "asc" ? "↑" : "↓") : ""}</th>
                <th on:click={() => sortTable("totalProfit")}>📈 Profit ($) {sortKey === "totalProfit" ? (sortOrder === "asc" ? "↑" : "↓") : ""}</th>
                <th on:click={() => sortTable("profitMargin")}>💹 Profit Margin (%) {sortKey === "profitMargin" ? (sortOrder === "asc" ? "↑" : "↓") : ""}</th>
                <th on:click={() => sortTable("region")}>🌍 Region {sortKey === "region" ? (sortOrder === "asc" ? "↑" : "↓") : ""}</th>
            </tr>
        </thead>
        <tbody>
            {#each sortedData as item}
                <tr>
                    <td>{item.productName}</td>
                    <td>{item.category}</td>
                    <td>{item.subCategory}</td>
                    <td>{item.segment}</td>
                    <td>${new Intl.NumberFormat().format(item.totalSales)}</td>
                    <td>{item.totalQuantity}</td>
                    <td>{item.avgDiscount}%</td>
                    <td>${new Intl.NumberFormat().format(item.totalProfit)}</td>
                    <td>{item.profitMargin}%</td>
                    <td>{item.region}</td>
                </tr>
            {/each}
        </tbody>
    </table>

    <!-- 📌 Empty State -->
    {#if sortedData.length === 0}
        <p class="empty-state">⚠️ No products match your search or selected filters.</p>
    {/if}

    <!-- 📌 Pagination Controls -->
    {#if totalPages > 1}
        <div class="pagination">
            <button on:click={prevPage} disabled={currentPage === 1}>⬅️ Prev</button>
            <span>Page {currentPage} of {totalPages}</span>
            <button on:click={nextPage} disabled={currentPage === totalPages}>Next ➡️</button>
        </div>
    {/if}
</div>

<!-- 📌 Styles -->
<style>
    .table-container {
        overflow-x: auto;
        background: white;
        padding: 1rem;
        border-radius: 12px;
        box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
    }

    table {
        width: 100%;
        border-collapse: collapse;
        font-size: 14px;
    }

    th, td {
        padding: 10px;
        border-bottom: 1px solid #ddd;
        text-align: left;
    }

    th {
        cursor: pointer;
        background: #f1f1f1;
        user-select: none;
    }

    th:hover {
        background: #e2e2e2;
    }

    .pagination {
        display: flex;
        justify-content: center;
        align-items: center;
        gap: 10px;
        margin-top: 10px;
    }

    .pagination button {
        padding: 6px 12px;
        border: none;
        background: #0077cc;
        color: white;
        border-radius: 5px;
        cursor: pointer;
        font-size: 14px;
    }

    .pagination button:disabled {
        background: #ccc;
        cursor: not-allowed;
    }

    .empty-state {
        text-align: center;
        font-size: 14px;
        color: #777;
        padding: 1rem;
    }

    @media (max-width: 768px) {
        table {
            font-size: 12px;
        }

        th, td {
            padding: 8px;
        }
    }
</style>
