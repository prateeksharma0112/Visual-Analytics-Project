<script>
    import BarChart from '../components/BarChart.svelte';
    import ScatterPlot from '../components/ScatterPlot.svelte';
    import LineChart from '../components/LineChart.svelte';
  
    const barData = [
      { category: "Furniture", sales: 7425 },
      { category: "Technology", sales: 13000 },
      { category: "Office Supplies", sales: 9200 },
    ];
  
    const scatterData = [
      { sales: 5000, profit: 1000 },
      { sales: 10000, profit: 2700 },
      { sales: 7500, profit: 2900 },
      { sales: 12000, profit: 3700 },
    ];
  
    const lineData = [
        { date: new Date(2023, 0, 1), sales: 5000 },
        { date: new Date(2023, 1, 1), sales: 7500 },
        { date: new Date(2023, 2, 1), sales: 8000 },
        { date: new Date(2023, 3, 1), sales: 6000 },
        { date: new Date(2023, 6, 1), sales: 12000 },
        { date: new Date(2023, 7, 1), sales: 14000 },
        { date: new Date(2023, 8, 1), sales: 12500 },
        { date: new Date(2023, 9, 1), sales: 15000 },
        { date: new Date(2023, 10, 1), sales: 17000 },
        { date: new Date(2023, 11, 1), sales: 18000 }
    ];
  </script>
  
  <style>
    main {
      font-family: 'Arial', sans-serif;
      background-color: #f7f7f7;
      color: #333;
      padding: 20px;
      margin: 0;
    }
  
    h1 {
      text-align: center;
      color: #2a2a2a;
      font-size: 2.5em;
      margin-bottom: 40px;
    }
  
    section {
      margin-bottom: 40px;
      padding: 20px;
      background-color: #fff;
      border-radius: 8px;
      box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
    }
    .chart-container {
      display: flex;
      justify-content: center;
      align-items: center;
      margin-top: 20px;
    }
    /* Responsive Design */
    @media (max-width: 768px) {
      .chart-container {
        flex-direction: column;
      }
    }
  </style>
  
  <main>
    <h1>Superstore Dashboard</h1>
  
    <section>
      <div class="chart-container">
        <BarChart {barData} />
      </div>
    </section>
  
    <section>
      <div class="chart-container">
        <ScatterPlot {scatterData} />
      </div>
    </section>
  
    <section>
      <div class="chart-container">
        <LineChart {lineData} />
      </div>
    </section>
  </main>
  