# postmanGeoJSON
A GeoJSON visual response viewer for Postman's Visualizer tool, built using [Leaflet](https://leafletjs.com/).

This Postman plugin was written for testing Ordnance Survey APIs which return a GeoJSON response, but it will work with any GeoJSON response from any service. It's based on the [GeoJSON Visualizer](https://www.postman.com/gold-meadow-42382/workspace/geojson-visualizer) plugin, with enhancements and better error handling.

*Please note that this plugin works with standard CRS84 GeoJSON only.*

## License
This code is published by Ordnance Survey under the [Open Government License](https://www.nationalarchives.gov.uk/doc/open-government-licence) (OGL).

## Contributors
- [@abiddiscombe](https://github.com/abiddiscombe)
- [@tmnnrs](https://github.com/tmnnrs)
- [@alex-matthew (base idea)](https://github.com/alex-mathew/Postman-GeoJSON-Visualizer-with-Fuzzy-Search).

## Installation
This plugin is designed to use the [OS Maps API](https://osdatahub.os.uk/docs/wmts/overview) to render a basemap if you provide an API key where specified in the code. If a key is not specified, the plugin will revert to OpenStreetMap tiles instead. The OS Maps API only covers Great Britain. If you're testing the response of an geospatial API with global coverage, you're probably best to use OSM tiles.

Copy the following code into the `Tests` component of your Postman project. Create a new request, click `Send`, and then set the response view to `Visualize` - a map will load within Postman showing a visual summary of the GeoJSON response. The total number of returned features is listed at the top of the frame.

```javascript
// Postman GeoJSON Plugin
// https://github.com/OrdnanceSurvey/postmanGeoJSON

const OS_MAPS_API_KEY = '' // leave blank for OSM tiles

const template = `
<link rel="stylesheet" href="https://unpkg.com/leaflet@1.7.1/dist/leaflet.css" />
<script src="https://unpkg.com/leaflet@1.7.1/dist/leaflet.js"></script>
<style>
    body { margin: 0; padding: 0; height: 100vh; display: grid; grid-template-rows: auto 1fr; }
    #top { padding: 6px; margin: 0; background: #f1f5f9; color: #1e293b; }
    #map { padding: 0; margin: 0; }
</style>
<div id="top"><small id="js-banner">Loading Response.</small></div>
<div id="map"></div>
<script>
    const map = L.map('map', { minZoom: 1, maxZoom: 18 });
    let apiKey = '${OS_MAPS_API_KEY}';
    let basemap = (!apiKey)
        ? L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
            attribution: '&copy; <a href="https://www.openstreetmap.org/copyright">OpenStreetMap</a> contributors'
        }).addTo(map)
        : L.tileLayer('https://api.os.uk/maps/raster/v1/zxy/Light_3857/{z}/{x}/{y}.png?key=' + apiKey, {
            maxZoom: 20,
            attribution: 'Contains OS data © Crown Copyright and Database Rights 2022'
        }).addTo(map)
    let overlay = L.geoJSON(null, {
        onEachFeature: function(feature, layer) {
            const properties = layer.feature.properties;
            let content = '<div">';
            for( let i in properties ) {
                content += '<h1 style="margin-top: 4px; margin-bottom: 4px; font-size: 1.4em; font-weight: bold;">' + i + '</h1><p style="margin-top: 2px; margin-bottom: 10px; padding-bottom: 4px; border-bottom: 1px solid #eee; font-size: 1em; font-family: monospace;">' + properties[i] + '</p>';
            }
            content += '</div>';
            layer.bindPopup(content, { maxHeight: 300, maxWidth: 1000 });
            layer.on({ mouseover: focusOnFeature, mouseout: resetOverlayStyle });
        },
        style: { weight: 1, color: '#FF1F5B', fillOpacity: 0.2 }
    });
    function focusOnFeature(e) { let t = e.target; t.setStyle({ weight: 3 }); t.bringToFront() }
    function resetOverlayStyle(e) { overlay.resetStyle(e.target) }
    pm.getData((error, data) => {
        if (!data.response.type) {
            document.getElementById('js-banner').innerText = "Error Parsing Response. It might not be GeoJSON.";
        } else {
            overlay.addData(data.response);
            overlay.addTo(map);
            map.fitBounds(overlay.getBounds());
            document.getElementById('js-banner').innerText = "Request has returned " + data.response.features.length.toString() + " unique features.";
        }
    })
</script>`; pm.visualizer.set(template, {response: pm.response.json()});
```
