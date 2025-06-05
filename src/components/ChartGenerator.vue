<script setup>
import { ref, onBeforeUnmount, nextTick } from 'vue';
import Papa from 'papaparse';
import { Chart, registerables } from 'chart.js/auto';
Chart.register(...registerables);

// --- Reactive Data ---
const parsedCsvData = ref(null);
const chartTitleInput = ref('Chart Title'); // Default title
const chartSourceInput = ref('Data Source');  // Default source

const chartCanvasRef = ref(null); // Ref for the canvas DOM element
let myChartInstance = null;     // To hold the Chart.js instance

// --- Helper: Define brand styles (using CSS variables) ---
// These assume your CSS variables are set in main.css
// For Chart.js, using var() directly in options works in modern browsers for many properties.
const delphiFontPrimary = 'var(--font-primary)';
const delphiFontSecondary = 'var(--font-secondary)';
const delphiHeadingColor = 'var(--color-heading)';
const delphiTextLightColor = 'var(--color-text-light)';
const delphiBorderMuteColor = 'var(--color-border-mute)';

// Define your brand color palette for charts (ensure these match your CSS vars or define directly)
// Using rgba for semi-transparent backgrounds, and rgb for solid borders
const delphiChartColorsRgba = [
  'rgba(255, 140, 105, 0.85)', // --delphi-primary (Peach/Orange)
  'rgba(91, 103, 255, 0.85)',  // --delphi-accent-blue
  'rgba(62, 182, 127, 0.85)',  // --delphi-accent-green
  'rgba(138, 43, 226, 0.85)', // --delphi-accent-purple
  'rgba(255, 192, 203, 0.85)', // Example Pink (replace or remove)
  'rgba(0, 128, 128, 0.85)'    // Example Teal (replace or remove)
];

const delphiChartColorsRgb = delphiChartColorsRgba.map(color => color.replace(/rgba\(([^,]+,[^,]+,[^,]+),[^)]+\)/, 'rgb($1)'));


// --- Methods ---
function handleFileUpload(event) {
  const file = event.target.files[0];
  if (file) {
    Papa.parse(file, {
      header: true,
      skipEmptyLines: true,
      complete: (results) => {
        parsedCsvData.value = results.data;
        console.log('Parsed CSV Data:', parsedCsvData.value);
        nextTick(() => { // Ensure DOM is updated before rendering
          renderChart();
        });
      },
      error: (error) => {
        console.error('Error parsing CSV:', error);
        parsedCsvData.value = null;
        if (myChartInstance) { myChartInstance.destroy(); myChartInstance = null; }
      }
    });
  } else {
    parsedCsvData.value = null;
    if (myChartInstance) { myChartInstance.destroy(); myChartInstance = null; }
  }
}

function renderChart() {
  if (!chartCanvasRef.value) {
    console.error('Canvas element not found!');
    return;
  }
  const ctx = chartCanvasRef.value.getContext('2d');

  if (myChartInstance) {
    myChartInstance.destroy();
    myChartInstance = null;
  }

  if (!parsedCsvData.value || parsedCsvData.value.length === 0) {
    // Clear canvas or show "No data" message
    ctx.clearRect(0, 0, chartCanvasRef.value.width, chartCanvasRef.value.height);
    ctx.font = `14px 'Geist'`; // Use actual font name
    ctx.fillStyle = '#141004'; // Use a more opaque text color for message
    ctx.textAlign = "center";
    ctx.fillText("No data to display. Please upload a CSV.", chartCanvasRef.value.width / 2, chartCanvasRef.value.height / 2);
    return;
  }

  let labels = [];
  let dataValues = [];
  // --- Your existing data preparation logic ---
  if (parsedCsvData.value[0] && 'Category' in parsedCsvData.value[0] && 'Value' in parsedCsvData.value[0]) {
    labels = parsedCsvData.value.map(row => row.Category).filter(label => label !== null && label !== undefined && label !== '');
    dataValues = parsedCsvData.value.filter(row => row.Category !== null && row.Category !== undefined && row.Category !== '').map(row => parseFloat(row.Value) || 0);
  } else if (parsedCsvData.value[0] && Object.keys(parsedCsvData.value[0]).length >= 2) {
    const keys = Object.keys(parsedCsvData.value[0]);
    labels = parsedCsvData.value.map(row => row[keys[0]]).filter(label => label !== null && label !== undefined && label !== '');
    dataValues = parsedCsvData.value.filter(row => row[keys[0]] !== null && row[keys[0]] !== undefined && row[keys[0]] !== '').map(row => parseFloat(row[keys[1]]) || 0);
  } else {
    console.error("CSV structure not suitable for basic chart.");
    return;
  }
  if (labels.length === 0 || dataValues.length === 0 || labels.length !== dataValues.length) {
    console.error("Data processing resulted in empty or mismatched labels/values arrays.");
    return;
  }
  // --- End Data Preparation ---

  // Delphi Intelligence Brand Colors (from your main.css)
  const diPrimary = '#FF8553';
  const diAccentBlue = '#5367FF';
  const diAccentGreen = '#49BB7E';
  const diAccentPurple = '#8A2BE2'; // Keep or replace
  // Add more of your actual brand palette here if needed
  const chartBarBackgroundColors = [
    `rgba(255, 133, 83, 0.85)`, // diPrimary with alpha
    `rgba(83, 103, 255, 0.85)`, // diAccentBlue with alpha
    `rgba(73, 187, 126, 0.85)`, // diAccentGreen with alpha
    `rgba(138, 43, 226, 0.85)`  // diAccentPurple with alpha
  ];
  const chartBarBorderColors = [
    diPrimary,
    diAccentBlue,
    diAccentGreen,
    diAccentPurple
  ];

  const headingColor = '#141004'; // --color-heading
  // For axis text and legend, let's use a more opaque version of your --color-text
  // Your --color-text (#14100470) is ~44% opacity. Let's try ~70-80% opacity for better readability.
  const axisTextColor = 'rgba(20, 16, 4, 0.75)'; // #141004 with 75% alpha
  // For grid lines, use a very subtle version of your text color or a light grey
  const gridLineColor = 'rgba(20, 16, 4, 0.1)'; // #141004 with 10% alpha - very subtle

  myChartInstance = new Chart(ctx, {
    type: 'bar',
    data: {
      labels: labels,
      datasets: [{
        label: 'Dataset', // You might hide this if only one dataset
        data: dataValues,
        backgroundColor: chartBarBackgroundColors,
        borderColor: chartBarBorderColors,
        borderWidth: 0, // Your reference images don't seem to have borders on bars
        borderRadius: 2, // Subtle rounding
        barPercentage: 0.6,
        categoryPercentage: 0.7
      }]
    },
    options: {
      responsive: true,
      maintainAspectRatio: false,
      layout: {
        padding: {
            top: 10, // More space if title is long
            right: 20, // Space for Y-axis labels if they are long
            bottom: 10,
            left: 10
        }
      },
      plugins: {
        title: {
          display: true,
          text: chartTitleInput.value || 'Chart Title',
          align: 'start',
          padding: {
            top: 0, // Align with very top of chart area
            bottom: 25 // Ample space below title
          },
          font: {
            family: 'BBModernPro', // Explicitly use the font name
            size: 18,        // Adjust based on reference
            weight: 'normal', // Your BBModernPro.otf is likely normal weight
          },
          color: headingColor
        },
        legend: {
          display: true, // Set to false if you usually have single datasets
          position: 'top',
          align: 'end',
          labels: {
            font: {
              family: 'Geist', // Explicit font name
              size: 11,
              weight: '400' // Normal weight for Geist
            },
            color: axisTextColor, // More opaque text
            boxWidth: 10,
            boxHeight: 10,
            padding: 15,
            usePointStyle: true,
            pointStyle: 'circle'
          }
        },
        tooltip: {
          enabled: true,
          backgroundColor: 'rgba(20, 16, 4, 0.9)', // Dark, almost opaque from your text color
          titleFont: { family: 'BBModernPro', size: 13, weight: 'normal' },
          bodyFont: { family: 'Geist', size: 12, weight: '400' },
          padding: 10,
          cornerRadius: 3,
          titleColor: '#FFFFFF', // White text on dark tooltip
          bodyColor: '#EFECE6', // Light beige text on dark tooltip
        }
      },
      scales: {
        x: {
          ticks: {
            font: { family: 'Geist', size: 10, weight: '400' },
            color: axisTextColor, // More opaque
            padding: 8
          },
          grid: {
            display: false, // No vertical grid lines
          },
          border: {
            display: true,
            color: gridLineColor, // Subtle axis line using the light text color
            width: 1
          }
        },
        y: {
          beginAtZero: true,
          grace: '10%', // Add some % padding to the top
          ticks: {
            font: { family: 'Geist', size: 10, weight: '400' },
            color: axisTextColor, // More opaque
            padding: 8,
          },
          grid: {
            display: true,
            color: gridLineColor, // Subtle dashed lines
            borderDash: [3, 3],
            drawBorder: false, // No solid Y axis line if grid is on
            // drawTicks: false, // Hides tick marks from the axis line
          },
          border: {
            display: false // Hide Y axis line itself
          }
        }
      }
    }
  });
}

// MAKE SURE THE onBeforeUnmount LIFECYCLE HOOK IS STILL PRESENT
onBeforeUnmount(() => {
  if (myChartInstance) {
    myChartInstance.destroy();
  }
});

</script>

<template>
  <div class="chart-generator-container">
    <section class="controls-section">
      <h2>1. Configure Your Chart</h2>
      
      <div class="control-group">
        <label for="csvFileInput">Upload CSV File:</label>
        <input type="file" id="csvFileInput" accept=".csv" @change="handleFileUpload" />
      </div>

      <div class="control-group">
        <label for="chartTitle">Chart Title:</label>
        <input type="text" id="chartTitle" placeholder="Enter chart title" v-model="chartTitleInput" @input="renderChart" />
      </div>

      <div class="control-group">
        <label for="chartSource">Source:</label>
        <input type="text" id="chartSource" placeholder="Enter data source" v-model="chartSourceInput" />
        <!-- Note: Source is typically displayed outside the chart, e.g., in App.vue footer -->
      </div>
      
      <button @click="renderChart" class="generate-button">Generate/Update Chart</button>
    </section>

        <section class="chart-display-section">
      <h2>2. Your Chart</h2>
      <div class="chart-wrapper">
        <div class="aspect-ratio-box"> 
          <canvas id="myGeneratedChart" ref="chartCanvasRef"></canvas>
        </div>
      </div>
      <!-- <button class="export-button">Export PNG</button> -->
    </section>
  </div>
</template>

<style scoped>
/* Styles specific to ChartGenerator.vue */
.chart-generator-container {
  display: flex;
  flex-direction: column;
  gap: 2rem; /* Space between controls and chart display */
  padding: 1rem; /* Some padding for the whole container */
}

@media (min-width: 992px) { /* Adjust breakpoint for wider layout */
  .chart-generator-container {
    flex-direction: row;
    gap: 2.5rem;
  }
  .controls-section {
    flex: 0 0 320px; /* Fixed width for controls, adjust as needed */
    max-width: 320px;
  }
  .chart-display-section {
    flex: 1; /* Takes up remaining space */
    min-width: 0; /* Allows flex item to shrink properly */
  }
}

.controls-section, .chart-display-section {
  padding: 1.5rem;
  background-color: #EFECE6; /* Fallback color */
  border-radius: 8px;
  box-shadow: 0 4px 12px rgba(0,0,0,0.07); /* Softer shadow */
}

h2 {
  margin-top: 0;
  margin-bottom: 1.5rem;
  font-family: var(--font-secondary, sans-serif);
  font-size: 1.25rem; /* Slightly smaller section titles */
  color: var(--color-heading, #333);
  border-bottom: 1px solid var(--color-border-mute, #eee);
  padding-bottom: 0.75rem;
}

.control-group {
  margin-bottom: 1.25rem;
}

.control-group label {
  display: block;
  margin-bottom: 0.5rem;
  font-family: var(--font-primary, sans-serif);
  font-weight: 500;
  font-size: 0.9rem;
  color: var(--color-text, #444);
}

.control-group input[type="text"],
.control-group input[type="file"] {
  width: 100%;
  padding: 0.65rem 0.85rem;
  border: 1px solid var(--color-border, #ccc);
  border-radius: 4px;
  box-sizing: border-box;
  background-color: var(--color-background-input, #fff);
  color: var(--color-text, #333);
  font-family: var(--font-primary, sans-serif);
  font-size: 0.95rem;
}

.control-group input[type="file"] {
    padding: 0.5rem; /* Specific padding for file input */
}


button.generate-button { /* More specific selector */
  display: block; /* Make button full width */
  width: 100%;
  padding: 0.85rem 1.5rem;
  background-color: var(--delphi-primary, #FF8C69);
  color: white;
  border: none;
  border-radius: 5px;
  cursor: pointer;
  font-family: var(--font-primary, sans-serif);
  font-weight: 600;
  font-size: 1rem;
  transition: background-color 0.2s ease-in-out, transform 0.1s ease;
  margin-top: 1rem;
}

button.generate-button:hover {
  background-color: var(--delphi-primary-dark, #E07B5A);
}
button.generate-button:active {
    transform: translateY(1px);
}


.chart-wrapper {
  position: relative;
  width: 100%; /* Take full width of its column */
  max-width: 1000px; /* Optional: Max width for very large screens */
  margin: 0 auto; /* Center if max-width is applied */
  /* background-color: var(--color-background); */ /* Set your desired background for the chart area */
  background-color: #FFFFFF; /* <<< Let's make it WHITE for now to match your reference images more closely */
  border-radius: 6px;
  /* border: 1px solid var(--color-border-mute, #eee); */ /* Optional */
  box-shadow: 0 4px 12px rgba(0,0,0,0.07); /* Add back shadow if removed */
  /* overflow: hidden; */ /* We might need to adjust this later */
}

.aspect-ratio-box {
  position: relative;
  width: 100%;
  padding-bottom: 56.25%; /* 9 / 16 * 100% = 56.25% (for 16:9) */
  height: 0;
  overflow: hidden; /* Important to contain the absolutely positioned canvas */
}

/* Make canvas fill the aspect-ratio-box */
#myGeneratedChart {
  position: absolute;
  top: 0;
  left: 0;
  width: 100% !important; /* Important for Chart.js responsiveness within the box */
  height: 100% !important; /* Important for Chart.js responsiveness within the box */
  display: block;
}
</style>