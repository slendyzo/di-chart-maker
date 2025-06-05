<script setup>
// We'll import Papa from 'papaparse' and Chart from 'chart.js/auto' later
// And use ref for reactive data
// import { ref } from 'vue';

// const chartData = ref(null);
// const chartInstance = ref(null);
// const chartCanvas = ref(null); // To get a reference to the canvas element

// function handleFileUpload(event) {
//   // Logic for file upload will go here
// }
</script>

<template>
  <div class="chart-generator-container">
    <section class="controls-section">
      <h2>1. Configure Your Chart</h2>
      
      <div class="control-group">
        <label for="csvFileInput">Upload CSV File:</label>
        <input type="file" id="csvFileInput" accept=".csv" /> <!-- @change="handleFileUpload" -->
      </div>

      <div class="control-group">
        <label for="chartTitle">Chart Title:</label>
        <input type="text" id="chartTitle" placeholder="Enter chart title" />
      </div>

      <div class="control-group">
        <label for="chartSource">Source:</label>
        <input type="text" id="chartSource" placeholder="Enter data source" />
      </div>
      
      <!-- More controls for chart type, axes, etc. will be added here -->

      <button>Generate Chart</button> <!-- We'll add functionality later -->
    </section>

    <section class="chart-display-section">
      <h2>2. Your Chart</h2>
      <div class="chart-wrapper">
        <canvas id="myGeneratedChart"></canvas> <!-- ref="chartCanvas" -->
      </div>
      <button>Export PNG</button> <!-- We'll add functionality later -->
    </section>
  </div>
</template>

<style scoped>
/* Styles specific to ChartGenerator.vue */
.chart-generator-container {
  display: flex;
  flex-direction: column;
  gap: 2rem; /* Space between controls and chart display */
}

@media (min-width: 768px) { /* Layout for wider screens */
  .chart-generator-container {
    flex-direction: row;
    gap: 3rem;
  }
  .controls-section {
    flex: 1; /* Takes up 1 part of the space */
  }
  .chart-display-section {
    flex: 2; /* Takes up 2 parts of the space, making it wider */
  }
}


.controls-section, .chart-display-section {
  padding: 1.5rem;
  background-color: var(--color-background-soft); /* We'll define this */
  border-radius: 8px;
  box-shadow: 0 2px 10px rgba(0,0,0,0.05);
}

h2 {
  margin-top: 0;
  margin-bottom: 1.5rem;
  color: var(--color-heading);
  border-bottom: 1px solid var(--color-border-mute);
  padding-bottom: 0.5rem;
}

.control-group {
  margin-bottom: 1rem;
}

.control-group label {
  display: block;
  margin-bottom: 0.5rem;
  font-weight: 500;
  color: var(--color-text);
}

.control-group input[type="text"],
.control-group input[type="file"] {
  width: 100%;
  padding: 0.75rem;
  border: 1px solid var(--color-border);
  border-radius: 4px;
  box-sizing: border-box; /* Important for padding and border */
  background-color: var(--color-background-input);
  color: var(--color-text);
}

button {
  padding: 0.75rem 1.5rem;
  background-color: var(--delphi-primary); /* Your brand primary color */
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  font-weight: 500;
  transition: background-color 0.2s ease-in-out;
}

button:hover {
  background-color: var(--delphi-primary-dark); /* A darker shade for hover */
}

.chart-wrapper {
  position: relative;
  min-height: 300px; /* Give some initial height */
  background-color: #fff; /* White background for the chart itself */
  padding: 1rem;
  border-radius: 4px;
  border: 1px solid var(--color-border-mute);
}

#myGeneratedChart {
  display: block;
  width: 100% !important; /* Important for Chart.js responsiveness */
  height: auto !important; /* Important for Chart.js responsiveness */
}
</style>