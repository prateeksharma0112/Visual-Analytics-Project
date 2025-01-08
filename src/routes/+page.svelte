<script>
    import Dataloader from "../components/Dataloader.svelte";
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
  </style>
  
  <main>
    <h1>Superstore Dashboard</h1>
    
    <section>
      <Dataloader />
    </section>
  </main>
  