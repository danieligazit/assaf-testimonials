<script>
    import { fly } from "svelte/transition";
    export let fact;
  </script>
  
  <style>
    .fact-box {
      background-color: #f4f4f4;
      border-left: 5px solid #3498db;
      box-shadow: 0px 4px 6px rgba(0, 0, 0, 0.1);
      margin: 16px 0;
      padding: 16px 20px;
      max-width: 450px;
      font-family: 'Arial', sans-serif;
    }
  
    .fact-box h3 {
      font-size: 20px;
      font-weight: 600;
      margin: 0;
      color: #2c3e50;
    }
  
    .fact-box p {
      font-size: 16px;
      line-height: 1.5;
      margin: 8px 0 0;
      color: #2c3e50;
    }
  </style>
  
  <div class="fact-box" in:fly="{{ y: -200, duration: 1000, delay: 200 }}">
    <h3>{fact.title}</h3>
    <p>{fact.description}</p>
  </div>
  