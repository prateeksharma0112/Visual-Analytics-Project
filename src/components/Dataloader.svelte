<script>
  import Papa from 'papaparse';
  import { onMount } from 'svelte';
  import { base } from '$app/paths';  // Added for GitHub Pages compatibility

  let data = [];

  // Load CSV data from the public folder using the correct path for GitHub Pages
  async function loadCSV() {
      try {
          const response = await fetch(`${base}/data/SuperstoreOrders.csv`);
          
          if (!response.ok) {
              throw new Error('Failed to fetch CSV data');
          }

          const csvText = await response.text();

          Papa.parse(csvText, {
              header: true, // Treat the first row as headers
              dynamicTyping: true, // Automatically convert data types
              complete: function(results) {
                  data = results.data;
                  console.log("Loaded Data:", data);
              },
              error: function(error) {
                  console.error("Parsing Error:", error);
              }
          });
      } catch (error) {
          console.error("Error fetching CSV:", error);
      }
  }

  // Load data on component mount
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
