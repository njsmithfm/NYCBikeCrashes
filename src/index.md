---
theme: deep-space
---

<div class="hero">
  <h1>NYC Bike Crash Dashboard</h1>

  <!-- <h2>Welcome to your new app! Edit&nbsp;<code style="font-size: 90%;">src/index.md</code> to change this page.</h2> -->
  <!-- <a href="https://observablehq.com/framework/getting-started">Get started<span style="display: inline-block; margin-left: 0.25rem;">↗︎</span></a> -->
</div>

<div id="map" style="width: 100%; height: 600px;"></div>

<script src="https://unpkg.com/leaflet/dist/leaflet.js"></script>
<script src="https://unpkg.com/leaflet.heat/dist/leaflet-heat.js"></script>
<link rel="stylesheet" href="https://unpkg.com/leaflet/dist/leaflet.css" />

<script>
  // Initialize the map centered on NYC Triborough view
  const map = L.map('map').setView([40.7128, -73.9], 11); // Centered on NYC with zoom level 11

  // Add a light mode tile layer (default OpenStreetMap)
  L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
    maxZoom: 70,
    attribution: '© OpenStreetMap contributors',
  }).addTo(map);

  // Fetch the data from the API
  fetch('https://data.cityofnewyork.us/resource/h9gi-nx95.json?$where=number_of_cyclist_injured%3E%3D1')
    .then(response => response.json())
    .then(data => {
      // Filter the data to only include records from the past year
      const oneYearAgo = new Date();
      oneYearAgo.setFullYear(oneYearAgo.getFullYear() - 1);

      // Prepare heatmap data
      let heatmapData = data
        .filter(item => new Date(item.crash_date) >= oneYearAgo && item.latitude && item.longitude)
        .map(item => [
          parseFloat(item.latitude),
          parseFloat(item.longitude),
          parseInt(item.number_of_cyclist_injured) || 1 // Default intensity to 1 if missing
        ]);

      // Create the heatmap layer
      // const heat = L.heatLayer(heatmapData, {
      //   radius: 35, // Increased radius for better overlap
      //   blur: 10,   // Slight blur for smooth transitions
      //   maxZoom: 17,
      //   minOpacity: 0.5, // Make heatmap more visible at all zoom levels
      //   gradient: {
      //     0.3: 'orange',
      //     0.7: 'red',
      //     1.0: 'darkred'
      //   },
      // }).addTo(map);

      // Add tooltips for each original point (not duplicates)
      data.forEach(item => {
        if (item.latitude && item.longitude) {
          const marker = L.circleMarker([item.latitude, item.longitude], {
            radius: 25, // Invisible marker
            blur: 1000,
            color: "orange", 
            maxZoom: 17,
            fillOpacity: 0.25,
            opacity: 0,
            minOpacity: 1, // Make heatmap more visible at all zoom levels
            gradient: {
              0.25: 'blue',
              0.5: 'lime',
              1.0: 'red'
            },
          }).addTo(map);

          marker.bindTooltip(
            `Cyclists Injured: ${item.number_of_cyclist_injured}<br>Date: ${new Date(item.crash_date).toLocaleDateString('en-US')}`,
            { direction: 'top', offset: [0, -10] }
          );
        }
      });
    });
</script>


```js
const csvURL = "https://data.cityofnewyork.us/resource/h9gi-nx95.csv?$where=number_of_cyclist_injured%3E=1"
fetch(csvURL)
  .then(response => {
    if (!response.ok) throw new Error(response.status);
    return response.text();
  });

let crashes = d3.csv(csvURL);

```
```js
const color = Plot.scale({
  color: {
    type: "categorical",
    domain: d3.groupSort(crashes, (D) => -D.length, (d) => d.borough),
    unknown: "var(--theme-foreground-muted)"
  }
});
```

<div class="hero">
  <h2>
  
  Reported bicycle crashes by borough: </h2>
<!-- Cards with big numbers -->
</div>
<div class="grid grid-cols-5">
  <div class="card">
    <h2>Brooklyn</h2>
    <span class="big">${crashes.filter((d) => d.borough === "BROOKLYN").length.toLocaleString("en-US")}</span>
  </div>
  <div class="card">
    <h2>Bronx</h2>
    <span class="big">${crashes.filter((d) => d.borough === "BRONX").length.toLocaleString("en-US")}</span>
  </div>
  <div class="card">
    <h2>Manhattan</h2>
     <span class="big">${crashes.filter((d) => d.borough === "MANHATTAN").length.toLocaleString("en-US")}</span>
  </div>
  <div class="card">
    <h2>Queens</h2>
     <span class="big">${crashes.filter((d) => d.borough === "QUEENS").length.toLocaleString("en-US")}</span>
  </div>
  <div class="card">
    <h2>Staten Island</h2>
     <span class="big">${crashes.filter((d) => d.borough === "STATEN ISLAND").length.toLocaleString("en-US")}</span>
  </div>
</div>