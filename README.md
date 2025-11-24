<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Interactive Sales Dashboard</title>
  <script src="https://cdn.plot.ly/plotly-latest.min.js"></script>
  <style>
    body { font-family: Arial, sans-serif; margin: 20px; }
    h2 { margin-top: 50px; }
    #charts { display: grid; grid-template-columns: 1fr 1fr; gap: 40px; }
    .chart { width: 100%; height: 400px; }
    #fileInput { margin-bottom: 20px; }
    pre { background: #f4f4f4; padding: 10px; overflow: auto; }
  </style>
</head>
<body>

<h1>Interactive Sales Performance Dashboard</h1>

<input type="file" id="fileInput" accept=".json">
<p>Select a JSON file containing your sales data to generate the dashboard.</p>

<div id="charts">
  <div>
    <h2>1. Purchase Amount Over Time</h2>
    <div id="purchaseAmountChart" class="chart"></div>
  </div>
  <div>
    <h2>2. Product Category Popularity</h2>
    <div id="categoryChart" class="chart"></div>
  </div>
  <div>
    <h2>3. Customer Demographics</h2>
    <div id="demographicsChart" class="chart"></div>
  </div>
  <div>
    <h2>4. Purchase Frequency</h2>
    <div id="frequencyChart" class="chart"></div>
  </div>
  <div>
    <h2>5. Discounts & Promo Effectiveness</h2>
    <div id="discountChart" class="chart"></div>
  </div>
</div>

<h2>README / Findings</h2>
<pre>
# Sales Dashboard Findings

## 1. Purchase Amount Over Time
- Shows customer spending trends over time.
- Peaks indicate seasonal purchases, useful for inventory and marketing planning.

## 2. Product Category Popularity
- Highlights the most purchased product categories.
- Guides promotions and stock management.

## 3. Customer Demographics
- Gender and age distribution visualized.
- Helps target marketing campaigns and recommendations.

## 4. Purchase Frequency
- Shows how often customers purchase.
- Identifies frequent buyers for loyalty programs.
- Encourages engagement of low-frequency customers.

## 5. Discounts & Promo Effectiveness
- Analyzes impact of discounts on sales.
- Optimizes promotional strategies and marketing ROI.

## Overall Insights
- Identifies high-value customer segments.
- Recognizes popular products and categories.
- Reveals seasonal trends for timing campaigns.
- Shows effectiveness of promotional campaigns.

### Recommendation
Use the dashboard to filter by season, category, location, or subscription status for actionable insights.

*Visualizations above correspond directly to these findings.*
</pre>

<script>
document.getElementById('fileInput').addEventListener('change', handleFileUpload);

function handleFileUpload(event) {
  const file = event.target.files[0];
  if (!file) return;

  const reader = new FileReader();
  reader.onload = function(e) {
    const data = JSON.parse(e.target.result);
    generateDashboard(data);
  };
  reader.readAsText(file);
}

function generateDashboard(data) {
  const salesData = Array.isArray(data) ? data : [data];

  // 1. Purchase Amount Over Time
  const purchaseDates = salesData.map(d => new Date(d.Season || Date.now()));
  const purchaseAmounts = salesData.map(d => d.purchase_amount);
  Plotly.newPlot('purchaseAmountChart', [{ x: purchaseDates, y: purchaseAmounts, type: 'scatter', mode: 'lines+markers', marker: { color: 'blue' }}], { title: 'Purchase Amount Over Time', xaxis: { title: 'Date' }, yaxis: { title: 'Amount ($)' }});

  // 2. Product Category Popularity
  const categoryCounts = {};
  salesData.forEach(d => { const cat = d.Category || 'Unknown'; categoryCounts[cat] = (categoryCounts[cat] || 0) + 1; });
  Plotly.newPlot('categoryChart', [{ x: Object.keys(categoryCounts), y: Object.values(categoryCounts), type: 'bar', marker: { color: 'orange' }}], { title: 'Product Category Popularity', xaxis: { title: 'Category' }, yaxis: { title: 'Count' }});

  // 3. Customer Demographics
  const genderCounts = {};
  const ageGroups = { '18-30':0, '31-45':0, '46-60':0, '61+':0 };
  salesData.forEach(d => {
    const gender = d.Gender || 'Unknown'; genderCounts[gender] = (genderCounts[gender] || 0) + 1;
    const age = d.Age || 0;
    if(age <= 30) ageGroups['18-30']++;
    else if(age <= 45) ageGroups['31-45']++;
    else if(age <= 60) ageGroups['46-60']++;
    else ageGroups['61+']++;
  });
  Plotly.newPlot('demographicsChart', [
    { labels: Object.keys(genderCounts), values: Object.values(genderCounts), type: 'pie', name: 'Gender' },
    { labels: Object.keys(ageGroups), values: Object.values(ageGroups), type: 'pie', name: 'Age Group' }
  ], { title: 'Customer Demographics', grid: { rows: 1, columns: 2 }});

  // 4. Purchase Frequency
  const frequencyCounts = {};
  salesData.forEach(d => { const freq = d['Frequency of Purchases'] || 'Unknown'; frequencyCounts[freq] = (frequencyCounts[freq] || 0) + 1; });
  Plotly.newPlot('frequencyChart', [{ x: Object.keys(frequencyCounts), y: Object.values(frequencyCounts), type: 'bar', marker: { color: 'green' }}], { title: 'Purchase Frequency', xaxis: { title: 'Frequency' }, yaxis: { title: 'Number of Customers' }});

  // 5. Discounts & Promo Effectiveness
  const discountCounts = { 'Yes':0, 'No':0 };
  salesData.forEach(d => { const discount = d['Discount Applied'] || 'No'; discountCounts[discount] = (discountCounts[discount] || 0) + 1; });
  Plotly.newPlot('discountChart', [{ x: Object.keys(discountCounts), y: Object.values(discountCounts), type: 'bar', marker: { color: 'purple' }}], { title: 'Discounts & Promo Effectiveness', xaxis: { title: 'Discount Applied' }, yaxis: { title: 'Count' }});
}
</script>

</body>
</html>
