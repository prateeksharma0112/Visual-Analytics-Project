<script>
  import Papa from 'papaparse';
  import { onMount } from 'svelte';

  export let data = [];
  export let onDataLoaded;

  onMount(async () => {
      const response = await fetch('data/SuperstoreOrders.csv');
      const csvText = await response.text();

      // Parsing CSV and ensuring proper number formatting
      Papa.parse(csvText, {
          header: true,
          dynamicTyping: true,
          complete: function(results) {
              data = results.data.map(row => ({
                  ...row,
                  Sales: Number(row.Sales) || 0,
                  Profit: Number(row.Profit) || 0
              }));
              console.log("Data Loaded:", data);
              onDataLoaded(data);
          }
      });
  });
</script>
