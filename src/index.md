<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>NYC Bicycle Crash Heatmap</title>
     <h1>Bicycle Injuries in NYC</h1>
     <br>
     <h4>This map shows geodata of vehicle collisions in the five boroughs which resulted in at least one cyclist getting injured. It suggests where there are cyclist-unsafe areas by expressing concentrations, as opposed to severity--meaning that what's emphasized are areas where crashes commonly occur, rather than individual crashes with high counts of cyclists injured. Vehicle data and a primary contributing factor are provided where available. <br>
     <br>
     Many thanks to NYC Open Data for providing the <a href="https://data.cityofnewyork.us/Public-Safety/Motor-Vehicle-Collisions-Crashes/h9gi-nx95/about_data">Motor Vehicle Collisions</a> dataset and to <a href="https://leafletjs.com/">Leaflet</a> and <a href="https://github.com/Leaflet/Leaflet.heat">Leaflet.heat</a> for the vizualization tools. 
      </h4>
     <br>
     <br>
  <!-- Leaflet CSS -->
  <link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css" />

  <!-- Leaflet JS -->
  <script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js"></script>

  <!-- Leaflet.heat JS -->
  <script src="https://unpkg.com/leaflet.heat/dist/leaflet-heat.js"></script>

  <style>
    /* Style for the map container */
    #map {
      width: 100%;
      height: 600px; /* Full-screen height */
    }
  </style>
</head>
<body>
  <!-- Map container -->
  <div id="map"></div>

  <script>
    // Initialize the Leaflet map
    const map = L.map('map').setView([40.7128, -73.9560], 12); // Centered on NYC

    // Add a base map layer (OpenStreetMap)
    L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
      maxZoom: 18,
      attribution: '© OpenStreetMap contributors'
    }).addTo(map);

    // Fetch crash data from NYC Open Data API
    fetch("https://data.cityofnewyork.us/resource/h9gi-nx95.json?$query=SELECT%0A%20%20%60crash_date%60%2C%0A%20%20%60crash_time%60%2C%0A%20%20%60borough%60%2C%0A%20%20%60zip_code%60%2C%0A%20%20%60latitude%60%2C%0A%20%20%60longitude%60%2C%0A%20%20%60location%60%2C%0A%20%20%60on_street_name%60%2C%0A%20%20%60off_street_name%60%2C%0A%20%20%60cross_street_name%60%2C%0A%20%20%60number_of_persons_injured%60%2C%0A%20%20%60number_of_persons_killed%60%2C%0A%20%20%60number_of_pedestrians_injured%60%2C%0A%20%20%60number_of_pedestrians_killed%60%2C%0A%20%20%60number_of_cyclist_injured%60%2C%0A%20%20%60number_of_cyclist_killed%60%2C%0A%20%20%60number_of_motorist_injured%60%2C%0A%20%20%60number_of_motorist_killed%60%2C%0A%20%20%60contributing_factor_vehicle_1%60%2C%0A%20%20%60contributing_factor_vehicle_2%60%2C%0A%20%20%60contributing_factor_vehicle_3%60%2C%0A%20%20%60contributing_factor_vehicle_4%60%2C%0A%20%20%60contributing_factor_vehicle_5%60%2C%0A%20%20%60collision_id%60%2C%0A%20%20%60vehicle_type_code1%60%2C%0A%20%20%60vehicle_type_code2%60%2C%0A%20%20%60vehicle_type_code_3%60%2C%0A%20%20%60vehicle_type_code_4%60%2C%0A%20%20%60vehicle_type_code_5%60%0AWHERE%20%60number_of_cyclist_injured%60%20%3E%200%0AORDER%20BY%20%60crash_date%60%20DESC%20NULL%20LAST")
      .then(response => response.json())
      .then(data => {

        data.forEach(item => {
        if (item.latitude && item.longitude) {
          const marker = L.circleMarker([item.latitude, item.longitude], {
            radius: 15, // Invisible marker
            blur: 0,
            maxZoom: 17,
            fillOpacity: 0,
            opacity: 0,
            minOpacity: 0, // Make heatmap more visible at all zoom levels
          }).addTo(map);

          marker.bindTooltip(
            `<b>Cyclists Injured:</b> ${item.number_of_cyclist_injured}<br>
            <b>Date:</b> ${new Date(item.crash_date).toLocaleDateString('en-US')}<br>
            <b>Vehicle 1:</b> ${item.vehicle_type_code1 || 'N/A'}<br>
            <b>Vehicle 2:</b> ${item.vehicle_type_code2 || 'N/A'}<br>
            <b>Contributing Factor:</b> ${item.contributing_factor_vehicle_1 || 'N/A'}`,
            { direction: 'top', 
              offset: [0, -10], 
              opacity: 0.83 
            }
          );
        }
      });
      

        // Process the data to extract heatmap points
        const heatData = data
          .filter(row => row.latitude && row.longitude) // Ensure rows have valid coordinates
          .map(row => {
            const lat = parseFloat(row.latitude); // Latitude
            const lng = parseFloat(row.longitude); // Longitude
            const injuries = parseInt(row.number_of_cyclist_injured || 0); // Number of cyclists injured

            // Set intensity to 1 if there's at least one cyclist injured, else 0.1
            let intensity = injuries > 0 ? 1 : 1;

            return [lat, lng, intensity]; // Return as [lat, lng, intensity]
          });

        // Add a heatmap layer to the map
        const heatLayer = L.heatLayer(heatData, {
          radius: 10,  // Increase radius to make points larger
          blur: 5,     // Reduce blur for sharper heatmap
          maxZoom: 18, // Max zoom level for heatmap intensity
          minOpacity: 0.45 // Minimum opacity for better visibility
        });

        // Add the heat layer to the map
        heatLayer.addTo(map);
      })
      .catch(error => {
        console.error("Error fetching or processing the data:", error);
      });

      
  </script>
</body>
</html>


```js
const jsonURL = "https://data.cityofnewyork.us/resource/h9gi-nx95.json?$query=SELECT%0A%20%20%60crash_date%60%2C%0A%20%20%60crash_time%60%2C%0A%20%20%60borough%60%2C%0A%20%20%60zip_code%60%2C%0A%20%20%60latitude%60%2C%0A%20%20%60longitude%60%2C%0A%20%20%60location%60%2C%0A%20%20%60on_street_name%60%2C%0A%20%20%60off_street_name%60%2C%0A%20%20%60cross_street_name%60%2C%0A%20%20%60number_of_persons_injured%60%2C%0A%20%20%60number_of_persons_killed%60%2C%0A%20%20%60number_of_pedestrians_injured%60%2C%0A%20%20%60number_of_pedestrians_killed%60%2C%0A%20%20%60number_of_cyclist_injured%60%2C%0A%20%20%60number_of_cyclist_killed%60%2C%0A%20%20%60number_of_motorist_injured%60%2C%0A%20%20%60number_of_motorist_killed%60%2C%0A%20%20%60contributing_factor_vehicle_1%60%2C%0A%20%20%60contributing_factor_vehicle_2%60%2C%0A%20%20%60contributing_factor_vehicle_3%60%2C%0A%20%20%60contributing_factor_vehicle_4%60%2C%0A%20%20%60contributing_factor_vehicle_5%60%2C%0A%20%20%60collision_id%60%2C%0A%20%20%60vehicle_type_code1%60%2C%0A%20%20%60vehicle_type_code2%60%2C%0A%20%20%60vehicle_type_code_3%60%2C%0A%20%20%60vehicle_type_code_4%60%2C%0A%20%20%60vehicle_type_code_5%60%0AWHERE%20%60number_of_cyclist_injured%60%20%3E%200%0AORDER%20BY%20%60crash_date%60%20DESC%20NULL%20LAST"
fetch(jsonURL)
  .then(response => {
    if (!response.ok) throw new Error(response.status);
    return response.text();
  });

let crashes = d3.json(jsonURL);

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
  
  Bicycle crashes by borough in the past year: </h2>
<!-- Cards with big numbers -->
</div>
<div class="grid grid-cols-4">
  <div class="grid-colspan-4 card"> 
    <h2>Brooklyn</h2>
    <span class="big">${crashes.filter((d) => d.borough === "BROOKLYN").length.toLocaleString("en-US")}</span>
  </div>
  <div class="grid-colspan-1 card">
    <h2>Bronx</h2>
    <span class="big">${crashes.filter((d) => d.borough === "BRONX").length.toLocaleString("en-US")}</span>
  </div>
  <div class="grid-colspan-1 card">
    <h2>Manhattan</h2>
     <span class="big">${crashes.filter((d) => d.borough === "MANHATTAN").length.toLocaleString("en-US")}</span>
  </div>
  <div class="grid-colspan-1 card">
    <h2>Queens</h2>
     <span class="big">${crashes.filter((d) => d.borough === "QUEENS").length.toLocaleString("en-US")}</span>
  </div>
  <div class="grid-colspan-1 card">
    <h2>Staten Island</h2>
     <span class="big">${crashes.filter((d) => d.borough === "STATEN ISLAND").length.toLocaleString("en-US")}</span>
  </div>
</div>
<br>
<br>

```js
// function sparkbar(max) {
//   return (x) => htl.html`<div style="
//     background: var(--theme-green);
//     color: black;
//     font: 10px/1.6 var(--sans-serif);
//     width: ${100 * x / max}%;
//     float: right;
//     padding-right: 3px;
//     box-sizing: border-box;
//     overflow: visible;
//     display: flex;
//     justify-content: end;">${x.toLocaleString("en-US")}`
// }

Inputs.table(crashes,{
//   format: {
//     number_of_cyclist_injured: sparkbar(d3.max(crashes, d => d.number_of_cyclist_injured))
//   },
  columns: [
    "borough",
    "crash_date",
    "contributing_factor_vehicle_1",
    "vehicle_type_code1",
    "vehicle_type_code2",
    "number_of_cyclist_injured"
  ],
  header: {
    number_of_cyclist_injured: "Cylists Injured",
    borough: "Borough",
    crash_date: "Date",
    vehicle_type_code1: "Vehicle 1",
    vehicle_type_code2: "Vehicle 2",
    contributing_factor_vehicle_1: "Contributing Factor"
  }
})
```