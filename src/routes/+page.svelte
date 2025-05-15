<h1>Bikes Map</h1>

<script>
    import mapboxgl from "mapbox-gl";
    import "../../node_modules/mapbox-gl/dist/mapbox-gl.css";

    mapboxgl.accessToken = "pk.eyJ1IjoiZ2Nhc3RpbGhvIiwiYSI6ImNtYXBoYXdyczAwZTgycW9nbmJpMXhhZmcifQ.tbm7x-lMF3mnZXemTnX2sA";

    import * as d3 from "d3";
    import { onMount } from "svelte";

    let stations = [];
    let map;

    let mapViewChanged = 0;
    
    $: map?.on("move", evt => mapViewChanged++);


    async function initMap() {
        map = new mapboxgl.Map({
            container: "map",
            center: [-71.09415, 42.36027],
            zoom: 12,
            style: "mapbox://styles/mapbox/streets-v12",
        });
        
        await new Promise(resolve => map.on("load", resolve));
        
        map.addSource("boston_route", {
            type: "geojson",
            data: "https://bostonopendata-boston.opendata.arcgis.com/datasets/boston::existing-bike-network-2022.geojson?outSR=%7B%22latestWkid%22%3A3857%2C%22wkid%22%3A102100%7D",
        });

        map.addLayer({
            id: "bikes_path_1", // A name for our layer (up to you)
            type: "line", // one of the supported layer types, e.g. line, circle, etc.
            source: "boston_route", // The id we specified in `addSource()`
            paint: {
                "line-color": "darkgreen"
            },
        });
        
        map.addSource("cambridge_route", {
            type: "geojson",
            data: "https://raw.githubusercontent.com/cambridgegis/cambridgegis_data/main/Recreation/Bike_Facilities/RECREATION_BikeFacilities.geojson",
        });

        map.addLayer({
            id: "bikes_path_2",
            type: "line",
            source: "cambridge_route",
            paint: {
                "line-color": "green"
            },
        });
    }

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

    onMount(() => {
        initMap();
        loadStationData();
    });

    function getCoords (station) {
        let point = new mapboxgl.LngLat(+station.Long, +station.Lat);
        let {x, y} = map.project(point);
        return {cx: x, cy: y};
    }

</script>

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
    }

    #map svg {
        opacity: 50%;
        position: absolute;
        z-index: 1;
        width: 100%;
        height: 100%;
        pointer-events: none;
    }
</style>