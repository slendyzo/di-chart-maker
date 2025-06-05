<script setup>
import { ref, onBeforeUnmount, nextTick } from 'vue';
import Papa from 'papaparse';
import { Chart, registerables } from 'chart.js/auto';
Chart.register(...registerables);

// --- Reactive Data ---
const parsedCsvData = ref(null);
const chartTitleInput = ref('Chart Title');
const chartSubtitleInput = ref('');
const chartSourceInput = ref('Default Source Text');
const chartCanvasRef = ref(null);
let myChartInstance = null;

// --- Color/Font Definitions ---
const headingColor = '#141004';
const axisTextColor = 'rgba(20, 16, 4, 0.75)';
const gridLineColor = 'rgba(20, 16, 4, 0.1)';
const subtitleColor = 'rgba(20, 16, 4, 0.9)';

const delphiChartColorsRgba = [
  'rgba(255, 140, 105, 0.85)', 'rgba(91, 103, 255, 0.85)',
  'rgba(62, 182, 127, 0.85)', 'rgba(138, 43, 226, 0.85)',
  'rgba(255, 192, 203, 0.85)', 'rgba(0, 128, 128, 0.85)'
];
const delphiChartColorsRgb = delphiChartColorsRgba.map(color => color.replace(/rgba\(([^,]+,[^,]+,[^,]+),[^)]+\)/, 'rgb($1)'));

// --- Custom Subtitle Plugin ---
const delphiSubtitlePlugin = {
  id: 'delphiSubtitle',
  afterDraw: (chart, args, options) => {
    if (!options.text || options.text.trim() === '') return;
    const { ctx } = chart; const title = chart.options.plugins.title;
    const subFontFamily = options.font?.family || 'Geist'; const subFontSize = options.font?.size || 13;
    const subFontWeight = options.font?.weight || 'normal'; const subFontStyle = options.font?.style || 'normal';
    const subFont = `${subFontStyle} ${subFontWeight} ${subFontSize}px ${subFontFamily}`;
    const subColor = options.color || subtitleColor; const subAlign = options.align || title.align || 'start';
    const subPaddingTop = options.padding?.top || 5;
    ctx.save(); ctx.font = subFont; ctx.fillStyle = subColor; ctx.textAlign = subAlign;
    let titleBlock = chart.titleBlock; let yPos;
    if (titleBlock && title.display) { yPos = titleBlock.bottom - title.padding.bottom + subPaddingTop + (subFontSize / 2);
    } else { yPos = (chart.options.layout.padding?.top || 0) + (title.font?.size || 18) + (title.padding?.bottom || 0) + subPaddingTop + (subFontSize / 2) ; }
    let xPos; const chartWidth = chart.chartArea.width; const canvasWidth = chart.canvas.width;
    const layoutPaddingLeft = chart.options.layout.padding?.left || 0; const layoutPaddingRight = chart.options.layout.padding?.right || 0;
    if (subAlign === 'start' || subAlign === 'left') { xPos = layoutPaddingLeft + (title.padding?.left || 0);
    } else if (subAlign === 'end' || subAlign === 'right') { xPos = canvasWidth - layoutPaddingRight - (title.padding?.right || 0);
    } else { xPos = layoutPaddingLeft + chartWidth / 2; }
    ctx.textBaseline = 'middle'; yPos += subFontSize * 0.2;
    ctx.fillText(options.text, xPos, yPos); ctx.restore();
  }
};

// --- Custom Source Text Plugin (with debug logs) ---
const delphiSourceTextPlugin = {
  id: 'delphiSourceText',
  afterDraw: (chart, args, options) => {
    console.log("[Source Plugin] Canvas dimensions: width=", chart.canvas.width, "height=", chart.canvas.height);
    console.log("[Source Plugin] afterDraw called. Raw options.text:", options.text);

    if (!options.text || options.text.trim() === '' || (options.text.startsWith("Source: ") && options.text.length <= "Source: ".length) ) {
      console.log("[Source Plugin] No valid text to draw or text is just 'Source: '.", "Full options.text was:", options.text);
      return;
    }
    const { ctx } = chart;
    const srcFontFamily = options.font?.family || 'Arial';
    const srcFontSize = options.font?.size || 30;
    const srcFontWeight = options.font?.weight || 'bold';
    const srcFontStyle = options.font?.style || 'normal';
    const srcFont = `${srcFontStyle} ${srcFontWeight} ${srcFontSize}px ${srcFontFamily}`;
    const srcColor = options.color || '#FF00FF';
    const paddingBottom = options.padding?.bottom || 50;
    const paddingLeft = chart.options.layout.padding?.left || 15;

    ctx.save();
    ctx.globalAlpha = 1;
    ctx.font = srcFont;
    ctx.fillStyle = srcColor;
    ctx.textAlign = 'left';
    ctx.textBaseline = 'bottom';
    const xPos = paddingLeft;
    const yPos = chart.canvas.height - paddingBottom;
    console.log(`[Source Plugin] Drawing "${options.text}" at x:${xPos}, y:${yPos} with font:${srcFont} color:${srcColor}`);
    ctx.fillText(options.text, xPos, yPos);
    ctx.restore();
  }
};

// --- Methods ---
function handleFileUpload(event) {
  const file = event.target.files[0];
  if (file) {
    Papa.parse(file, {
      header: true, skipEmptyLines: true,
      complete: (results) => {
        console.log("[FileUpload] Papa.parse complete. Results data:", results.data); // DEBUG
        parsedCsvData.value = results.data;
        console.log("[FileUpload] parsedCsvData.value set to:", parsedCsvData.value); // DEBUG
        nextTick(() => { renderChart(); });
      },
      error: (error) => { console.error('Error parsing CSV:', error); parsedCsvData.value = null; if (myChartInstance) { myChartInstance.destroy(); myChartInstance = null; } }
    });
  } else { parsedCsvData.value = null; if (myChartInstance) { myChartInstance.destroy(); myChartInstance = null; } }
}

function renderChart() {
  console.log("[RenderChart] Called. chartCanvasRef available?", !!chartCanvasRef.value); // DEBUG
  if (!chartCanvasRef.value) { console.error('[RenderChart] Canvas element not found!'); return; }
  const ctx = chartCanvasRef.value.getContext('2d');
  if (myChartInstance) { myChartInstance.destroy(); myChartInstance = null; }

  console.log("[RenderChart] parsedCsvData.value before check:", parsedCsvData.value); // DEBUG
  if (!parsedCsvData.value || parsedCsvData.value.length === 0) {
    console.log("[RenderChart] Condition met: No data to display or data is empty."); // DEBUG
    ctx.clearRect(0, 0, chartCanvasRef.value.width, chartCanvasRef.value.height);
    ctx.font = `14px 'Geist'`; ctx.fillStyle = '#141004'; ctx.textAlign = "center";
    ctx.fillText("No data to display. Please upload a CSV.", chartCanvasRef.value.width / 2, chartCanvasRef.value.height / 2);
    return;
  }

  console.log("[RenderChart] Data seems okay, proceeding to prepare labels/values."); // DEBUG
  let labels = []; let dataValues = [];
  if (parsedCsvData.value[0] && 'Category' in parsedCsvData.value[0] && 'Value' in parsedCsvData.value[0]) { labels = parsedCsvData.value.map(r => r.Category).filter(l => l!=null && l!==''); dataValues = parsedCsvData.value.filter(r => r.Category!=null && r.Category!=='').map(r => parseFloat(r.Value)||0);
  } else if (parsedCsvData.value[0] && Object.keys(parsedCsvData.value[0]).length >= 2) { const keys = Object.keys(parsedCsvData.value[0]); labels = parsedCsvData.value.map(r => r[keys[0]]).filter(l => l!=null && l!==''); dataValues = parsedCsvData.value.filter(r => r[keys[0]]!=null && r[keys[0]]!=='').map(r => parseFloat(r[keys[1]])||0);
  } else { console.error("CSV structure not suitable."); return; }
  
  console.log("[RenderChart] Labels prepared:", labels); // DEBUG
  console.log("[RenderChart] DataValues prepared:", dataValues); // DEBUG

  if (labels.length === 0 || dataValues.length === 0 || labels.length !== dataValues.length) {
    console.error("[RenderChart] Data processing resulted in empty or mismatched labels/values arrays after prep."); // DEBUG
    // Optionally display error on canvas
    ctx.clearRect(0, 0, chartCanvasRef.value.width, chartCanvasRef.value.height);
    ctx.font = `14px 'Geist'`; ctx.fillStyle = 'red'; ctx.textAlign = "center";
    ctx.fillText("Error processing data for chart.", chartCanvasRef.value.width / 2, chartCanvasRef.value.height / 2);
    return;
  }

  myChartInstance = new Chart(ctx, {
    type: 'bar',
    data: { labels: labels, datasets: [{ label: 'Dataset', data: dataValues, backgroundColor: delphiChartColorsRgba, borderColor: delphiChartColorsRgb, borderWidth: 0, borderRadius: 2, barPercentage: 0.6, categoryPercentage: 0.7 }] },
    plugins: [delphiSubtitlePlugin, delphiSourceTextPlugin],
    options: {
      responsive: true, maintainAspectRatio: false,
      layout: { padding: { top: 20, right: 25, bottom: 80, left: 35 } }, // Increased bottom padding for test
      plugins: {
        title: { display: true, text: chartTitleInput.value || 'Default Chart Title', align: 'start', padding: { top: 5, bottom: 5 }, font: { family: 'BBModernPro', size: 24, weight: 'normal' }, color: headingColor },
        delphiSubtitle: { text: chartSubtitleInput.value, align: 'start', padding: { top: 4 }, font: { family: 'Geist', size: 13, weight: 'normal' }, color: subtitleColor },
        delphiSourceText: {
          text: chartSourceInput.value && chartSourceInput.value.trim() !== '' ? `Source: ${chartSourceInput.value}` : '',
          font: { family: 'Arial', size: 30, weight: 'bold' }, // TEST VALUES
          color: '#FF00FF', // TEST VALUE (Magenta)
          padding: { bottom: 50 } // TEST VALUE (increased padding from bottom)
        },
        legend: { display: true, position: 'top', align: 'end', labels: { font: { family: 'Geist', size: 11, weight: '400' }, color: axisTextColor, boxWidth: 10, boxHeight: 10, padding: 15, usePointStyle: true, pointStyle: 'circle'} },
        tooltip: { enabled: true, backgroundColor: 'rgba(20, 16, 4, 0.9)', titleFont: { family: 'BBModernPro', size: 13, weight: 'normal' }, bodyFont: { family: 'Geist', size: 12, weight: '400' }, padding: 10, cornerRadius: 3, titleColor: '#FFFFFF', bodyColor: '#EFECE6',}
      },
      scales: {
        x: { ticks: { font: { family: 'Geist', size: 10, weight: '400' }, color: axisTextColor, padding: 8 }, grid: { display: false }, border: { display: true, color: gridLineColor, width: 1 } },
        y: { beginAtZero: true, grace: '10%', ticks: { font: { family: 'Geist', size: 10, weight: '400' }, color: axisTextColor, padding: 8 }, grid: { display: true, color: gridLineColor, borderDash: [3, 3], drawBorder: false }, border: { display: false } }
      }
    }
  });
}

async function exportChartImage() { /* ... same ... */ }
onBeforeUnmount(() => { if (myChartInstance) { myChartInstance.destroy(); } });
</script>

<template>
  <!-- ... Your template with @input="renderChart" on all relevant inputs ... -->
  <div class="chart-generator-container">
    <section class="controls-section">
      <h2>1. Configure Your Chart</h2>
      <div class="control-group"><label for="csvFileInput">Upload CSV File:</label><input type="file" id="csvFileInput" accept=".csv" @change="handleFileUpload" /></div>
      <div class="control-group"><label for="chartTitle">Chart Title:</label><input type="text" id="chartTitle" placeholder="Enter chart title" v-model="chartTitleInput" @input="renderChart" /></div>
      <div class="control-group"><label for="chartSubtitle">Chart Subtitle (Optional):</label><input type="text" id="chartSubtitle" placeholder="Enter chart subtitle" v-model="chartSubtitleInput" @input="renderChart" /></div>
      <div class="control-group"><label for="chartSource">Source:</label><input type="text" id="chartSource" placeholder="Enter data source" v-model="chartSourceInput" @input="renderChart" /></div>
      <button @click="renderChart" class="generate-button">Generate/Update Chart</button>
    </section>
    <section class="chart-display-section">
      <h2>2. Your Chart</h2>
      <div class="chart-wrapper">
        <div class="aspect-ratio-box"> 
          <canvas id="myGeneratedChart" ref="chartCanvasRef"></canvas>
          <img src="/chart_overlay_16x9.png" alt="Chart Branding Overlay" class="chart-branding-overlay" />
        </div>
      </div>
      <button @click="exportChartImage" class="export-button">Export Chart as PNG</button>
    </section>
  </div>
</template>

<style scoped>
    /* ... Your styles ... */
    .chart-generator-container { display: flex; flex-direction: column; gap: 2rem; padding: 1rem; }
    @media (min-width: 992px) { .chart-generator-container { flex-direction: row; gap: 2.5rem; } .controls-section { flex: 0 0 320px; max-width: 320px; } .chart-display-section { flex: 1; min-width: 0; } }
    .controls-section, .chart-display-section { padding: 1.5rem; background-color: var(--chart-card-background, #FFF); border-radius: 8px; box-shadow: 0 4px 12px rgba(0,0,0,0.07); }
    h2 { margin-top: 0; margin-bottom: 1.5rem; font-family: var(--font-secondary, sans-serif); font-size: 1.25rem; color: var(--color-heading, #333); border-bottom: 1px solid var(--color-border-mute, #eee); padding-bottom: 0.75rem; }
    .control-group { margin-bottom: 1.25rem; }
    .control-group label { display: block; margin-bottom: 0.5rem; font-family: var(--font-primary, sans-serif); font-weight: 500; font-size: 0.9rem; color: var(--color-text, #444); }
    .control-group input[type="text"], .control-group input[type="file"] { width: 100%; padding: 0.65rem 0.85rem; border: 1px solid var(--color-border, #ccc); border-radius: 4px; box-sizing: border-box; background-color: var(--color-background-input, #fff); color: var(--color-text, #333); font-family: var(--font-primary, sans-serif); font-size: 0.95rem; }
    .control-group input[type="file"] { padding: 0.5rem; }
    button.generate-button, button.export-button { display: block; width: 100%; padding: 0.85rem 1.5rem; background-color: var(--delphi-primary, #FF8C69); color: white; border: none; border-radius: 5px; cursor: pointer; font-family: var(--font-primary, sans-serif); font-weight: 600; font-size: 1rem; transition: background-color 0.2s ease-in-out, transform 0.1s ease; margin-top: 1rem; }
    button.generate-button:hover, button.export-button:hover { background-color: var(--delphi-primary-dark, #E07B5A); }
    button.generate-button:active, button.export-button:active { transform: translateY(1px); }
    button.export-button { background-color: var(--delphi-accent-green, #49BB7E); margin-top: 0.5rem;}
    button.export-button:hover { background-color: #3E9D6A; }
    .chart-wrapper { position: relative; width: 100%; max-width: 1000px; margin: 0 auto; background-color: var(--chart-drawing-area-background, #EFECE6); border-radius: 6px; box-shadow: 0 4px 12px rgba(0,0,0,0.07); }
    .aspect-ratio-box { position: relative; width: 100%; padding-bottom: 56.25%; height: 0; overflow: hidden; }
    #myGeneratedChart { position: absolute; top: 0; left: 0; width: 100% !important; height: 100% !important; display: block; z-index: 1; }
    .chart-branding-overlay { position: absolute; top: 0; left: 0; width: 100%; height: 100%; pointer-events: none; z-index: 2; display: block; }
</style>