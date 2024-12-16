<div id="map" style="width: 100%; height: 500px;"></div>

<script src="https://unpkg.com/leaflet/dist/leaflet.js"></script>
<script src="https://unpkg.com/leaflet.heat/dist/leaflet-heat.js"></script>
<link rel="stylesheet" href="https://unpkg.com/leaflet/dist/leaflet.css" />

<script>
  // Initialize the map centered on Brooklyn
  const map = L.map('map').setView([40.6782, -73.9442], 12); // Zoom level 12 for Brooklyn

  // Add a light mode tile layer (default OpenStreetMap)
  L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
    maxZoom: 18,
    attribution: '© OpenStreetMap contributors',
  }).addTo(map);

  // Fetch the data from the API
  fetch('https://data.cityofnewyork.us/resource/h9gi-nx95.json?$where=number_of_cyclist_injured>0')
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

      // Duplicate and randomize data to simulate higher density for testing
      const randomize = (val) => val + (Math.random() - 0.5) * 0.002; // Small random offset
      heatmapData = heatmapData.flatMap(point => {
        const [lat, lon, intensity] = point;
        return [
          point,
          [randomize(lat), randomize(lon), intensity],
          [randomize(lat), randomize(lon), intensity]
        ];
      });

      // Create the heatmap layer
      const heat = L.heatLayer(heatmapData, {
        radius: 35, // Increased radius for better overlap
        blur: 20,   // Slight blur for smooth transitions
        maxZoom: 17,
        minOpacity: 0.5, // Make heatmap more visible at all zoom levels
        gradient: {
          0.3: 'yellow',
          0.5: 'orange',
          0.7: 'red',
          1.0: 'darkred'
        },
      }).addTo(map);

      // Add tooltips for each original point (not duplicates)
      data.forEach(item => {
        if (item.latitude && item.longitude) {
          const marker = L.circleMarker([item.latitude, item.longitude], {
            radius: 0, // Invisible marker
            fillOpacity: 0,
            opacity: 0
          }).addTo(map);

          marker.bindTooltip(
            `Injuries: ${item.number_of_cyclist_injured}<br>Date: ${item.crash_date}`,
            { direction: 'top', offset: [0, -10] }
          );
        }
      });
    });
</script>
