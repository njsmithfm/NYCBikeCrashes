---
title: Dynamic API Test
---

<!DOCTYPE html>
<html lang="en">
<head width=100%>

  <!-- Leaflet CSS -->
  <link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css" />

  <!-- Leaflet JS -->
  <script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js"></script>

  <!-- Leaflet.heat JS -->
  <script src="https://unpkg.com/leaflet.heat/dist/leaflet-heat.js"></script>

</head>

<body>
  <!-- Map container -->
  <div id="map"></div>
  <style>
    /* Style for the map container */
    p, table, figure, figcaption, h1, h2, h3, h4, h5, h6, .katex-display {
      max-width: 100%;
    }
    #map {
      width: 100%;
      height: 600px; /* Full-screen height */
    }
  </style>

<script>
  // Initialize Leaflet map
  const map = L.map('map',{
    zoom: 12,
    maxZoom: 16,                
    minZoom: 10  
  }
  ).setView([40.7128, -73.9560], 12); // Centered on NYC
  // Add a base map layer (OpenStreetMap)
  L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
    maxZoom: 18,
    attribution: '&copy; <a href="https://www.openstreetmap.org/copyright">OpenStreetMap</a> contributors'
  }).addTo(map);

  // Select Input to change the view of the map
  const MapView = view(Inputs.select(x11colors, {value: "steelblue", label: "Favorite color"}));
  Mapview;
  
  //Add the Dynamic API
  const api_url = 'https://data.cityofnewyork.us/resource/h9gi-nx95.json?$where=number_of_cyclist_injured%3E%3D1';
  // 22 Jan 2025 This API is filtered for cyclists injured >= 1
  // still need to filter further and sort by date descending

  
  async function getDynamicAPI() {
    const response = await fetch(api_url);
    const data = await response.json();
    console.log(data)
  }

  getDynamicAPI();



</script>
