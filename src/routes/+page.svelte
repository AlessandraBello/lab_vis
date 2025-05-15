<script>
import mapboxgl from "mapbox-gl";
import "../../node_modules/mapbox-gl/dist/mapbox-gl.css";
import * as d3 from "d3";
import { onMount } from "svelte";
    
mapboxgl.accessToken = "pk.eyJ1IjoiYWxlYmVsbG8iLCJhIjoiY21hcGgzOWs5MDBkYzJxcHZlOHJrNXIxZyJ9.g6pAhHS0puISaSDjG1H7ng";

let map;
let stations = [];
let mapViewChanged = 0;

async function loadStationData() {
    try {
        const csvUrl = 'https://vis-society.github.io/labs/8/data/bluebikes-stations.csv';
        const data = await d3.csv(csvUrl);
        
        stations = data.map(station => ({
            id: station.Number,
            name: station.NAME,
            Lat: +station.Lat,
            Long: +station.Long,
        }));
        // console.log('Stations loaded:', stations.length);
    } catch (error) {
        console.error('Error loading station data:', error);
    }
}


async function initMap() {
	map = new mapboxgl.Map({
		container: 'map',
		center: [-71.09415, 42.36027],
		zoom: 12,
		style: "mapbox://styles/mapbox/streets-v12",
	});
	await new Promise(resolve => map.on("load", resolve));
	map.addSource("cambridge_route", {
		type: "geojson",
		data: "https://bostonopendata-boston.opendata.arcgis.com/datasets/boston::existing-bike-network-2022.geojson?outSR=%7B%22latestWkid%22%3A3857%2C%22wkid%22%3A102100%7D",
	});
    map.addSource("boston_route", {
		type: "geojson",
		data: "https://raw.githubusercontent.com/cambridgegis/cambridgegis_data/main/Recreation/Bike_Facilities/RECREATION_BikeFacilities.geojson",
	});
    map.addLayer({
        id: "SOME_ID", // A name for our layer (up to you)
        type: "line", // one of the supported layer types, e.g. line, circle, etc.
        source: "boston_route", // The id we specified in `addSource()`
        paint: {
            "line-color": "green",
            "line-width": 3,
            "line-opacity": 0.4
        },
    });
}

function getCoords (station) {
	let point = new mapboxgl.LngLat(station.Long, station.Lat);
	let {x, y} = map.project(point);
	return {cx: x, cy: y};
}


onMount(() => {
	initMap();
    loadStationData();
});

$: map?.on("move", evt => mapViewChanged++);

</script>

<h1>Lab 8</h1>

<div id="map">
    <svg>
        {#key mapViewChanged}
            {#each stations as station}
                <circle { ...getCoords(station) } r="5" fill="steelblue" />
            {/each}
        {/key}
    </svg>
</div>

<style>
@import url("$lib/global.css");

#map {
	flex: 1;
    background-color: red;
}

#map svg {
    z-index: 1;
    position: absolute;
    width: 100%;
    height: 100%;
    pointer-events: none;
}
</style>
