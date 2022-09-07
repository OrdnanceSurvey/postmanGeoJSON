# postmanGeoJSON
A GeoJSON visual response viewer for Postman's Visualizer tool, built using [Leaflet](https://leafletjs.com/).

This Postman plugin was written for testing Ordnance Survey APIs, which return a GeoJSON response, but it will work with any GeoJSON response from any service. It's based on the [GeoJSON Visualizer](https://www.postman.com/gold-meadow-42382/workspace/geojson-visualizer) plugin, with various enhancements.

Please note that this plugin is designed to work with standard CRS84 GeoJSON geometry only.

## License
This code is shared by Ordnance Survey under the Open Government License (OGL). For more information, [click here](https://www.nationalarchives.gov.uk/doc/open-government-licence).

## Contributors
- [@gold-meadow-42382](https://www.postman.com/gold-meadow-42382/workspace/geojson-visualizer/overview) (Original project version)
- [@tmnnrs](https://github.com/tmnnrs)
- [@abiddiscombe](https://github.com/abiddiscombe)

## Installation
This plugin is designed to use the [OS Maps API](https://osdatahub.os.uk/docs/wmts/overview) to render a basemap. If you don't provide an API key where specified, the plugin will revert to OpenStreetMap tiles instead.

> NOTE  
> The OS Maps API only covers Great Britain. If you're testing the response of an API with global coverage, you're probably best to use OSM tiles instead.

Copy the following code into the `Tests` component of your Postman project, and change any values as required. Create a new request, click `Send`, and then set the response view to `Visualize` - a map should then load within Postman. The total number of returned features is listed at the top of the frame.

```javascript
// postmanGeoJSON

let template = `

<!-- leaflet.js -->
<link rel="stylesheet" href="https://unpkg.com/leaflet@1.7.1/dist/leaflet.css" />
<script src="https://unpkg.com/leaflet@1.7.1/dist/leaflet.js"></script>
<!-- end -->

<style>
    body { margin: 0; padding: 0; height: 100vh; display: grid; grid-template-rows: auto 1fr; }
    #top { padding: 6px; margin: 0; background: #f9f9f9; color: #333333; }
    #map { padding: 0; margin: 0; }
</style>

<div id="top">
    <small>Request has returned <span id="js-featuresReturned"></span> unique feature(s).</small>
</div>

<div id="map">
    <noscript>
        <p>JavaScript Required!</p>
        <p>Please enable JavaScript to run this app.</p>
    </noscript>
</div>

<script>

    // OS MAPS API KEY
    // leave this string empty to enable OSM tiles
    const apiKey = '';

    const map = L.map('map', {
        minZoom: 7,
        maxZoom: 20,
        center: [ 50.727589, -3.541809 ],
        zoom: 6
    });

    let basemap, overlay;

    if (!apiKey) {
        basemap = L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
            attribution: '&copy; <a href="https://www.openstreetmap.org/copyright">OpenStreetMap</a> contributors'
        });
    } else {
        basemap = L.tileLayer('https://api.os.uk/maps/raster/v1/zxy/Light_3857/{z}/{x}/{y}.png?key=' + apiKey, {
            maxZoom: 20,
            attribution: 'Contains OS data © Crown copyright and database rights 2022'
        });
    }

    basemap.addTo(map);

    overlay = L.geoJSON(null, {
        onEachFeature: function(feature, layer) {
            const properties = layer.feature.properties;
            let content = '<table style="font-size:1em">';
            for( let i in properties ) {
                content += '<tr><td>' + i + '</td><td>' + properties[i] + '</td></tr>';
            }
            content += '</table>';
            layer.bindPopup(content, { maxHeight: 300, maxWidth: 800 });
        },
        style: {
            weight: 1,
            color: '#FF1F5B',
            fillOpacity: 0.2
        }
    });

    pm.getData( function (error, data) {
        overlay.addData(data.response);
        overlay.addTo(map);
        map.fitBounds(overlay.getBounds());
        document.getElementById('js-featuresReturned').innerText = data.response.features.length;
    })

</script>
`;

pm.visualizer.set(template, {response: pm.response.json()});
```
