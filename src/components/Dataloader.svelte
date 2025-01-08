<script>
  import Papa from 'papaparse';
  import { onMount } from 'svelte';

  let data = [];

  // Load CSV data from the public folder
  async function loadCSV() {
      const response = await fetch('/data/SuperstoreOrders.csv');
      const csvText = await response.text();

      Papa.parse(csvText, {
          header: true, // Treat the first row as headers
          dynamicTyping: true, // Convert data types automatically
          complete: function(results) {
              data = results.data;
              console.log("Loaded Data:", data);
          }
      });
  }

  onMount(() => {
      loadCSV();
  });
</script>

<h1>Superstore Sales Data</h1>
{#if data.length > 0}
  <p>Data loaded successfully!</p>
  <p>Showing first 5 records:</p>
  <ul>
      {#each data.slice(0, 5) as record (record['Row ID'])}
          <li>{record['Order ID']} - {record['Product Name']}</li>
      {/each}
  </ul>
{:else}
  <p>Loading data...</p>
{/if}
