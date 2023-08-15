<template>
    <div id="map">
    </div>
    <div class="container">
        <div class="mb-3">
            <label for="latitude" class="form-label">Latitude</label>
            <input type="text" class="form-control" id="latitude" v-model="input_latitude">
        </div>
        <div class="mb-3">
            <label for="longitude" class="form-label">Longitude</label>
            <input type="text" class="form-control" id="longitude" v-model="input_langitude">
        </div>
        <button type="button" class="btn btn-success" @click="addLocation">Add</button>
    </div>
</template>
  
<script>
import Map from 'ol/Map';
import View from 'ol/View';
import TileLayer from 'ol/layer/Tile';
import OSM from 'ol/source/OSM';
import VectorSource from 'ol/source/Vector';
import VectorLayer from 'ol/layer/Vector';
import Feature from 'ol/Feature';
import Point from 'ol/geom/Point';
import { Icon, Style } from 'ol/style';
import { v4 as uuidv4 } from 'uuid';
import { fromLonLat } from 'ol/proj';

const locations = [
    {
        id: uuidv4(),
        name: 'Lokasi A',
        latitude: -7.7864,
        longitude: 110.3658
    },
    {
        id: uuidv4(),
        name: 'Lokasi B',
        latitude: -7.8015,
        longitude: 110.3672
    },
    {
        id: uuidv4(),
        name: 'Lokasi C',
        latitude: -7.7937,
        longitude: 110.3708
    }
];

export default {
    name: 'MapView',
    data() {
        return {
            input_latitude: "",
            input_langitude: "",
        }
    },

    mounted() {
        this.initializeMap();
        this.addMarkers();
    },
    methods: {
        initializeMap() {
            const map = new Map({
                target: 'map',
                layers: [
                    new TileLayer({
                        source: new OSM()
                    })
                ],
                view: new View({
                    center: fromLonLat([110.3695, -7.7956]),
                    zoom: 10
                })
            });

            this.map = map;
        },

        addMarkers() {
            const vectorSource = new VectorSource();
            const vectorLayer = new VectorLayer({
                source: vectorSource
            });

            locations.forEach(location => {
                const iconFeature = new Feature({
                    geometry: new Point(fromLonLat([location.longitude, location.latitude])),
                    name: location.name,
                    draggable: true
                });

                const iconStyle = new Style({
                    image: new Icon({
                        src: './marker.png',
                        scale: 0.1
                    })
                });

                iconFeature.setStyle(iconStyle);
                vectorSource.addFeature(iconFeature);
            });


            this.map.addLayer(vectorLayer);
        },

        addLocation() {
            if (this.input_latitude != "" && this.input_langitude != "") {
                locations.push({
                    id: uuidv4(),
                    name: `Lokasi ${locations.length + 1}`,
                    latitude: this.input_latitude,
                    longitude: this.input_langitude

                })
            }

            this.addMarkers()

            this.input_latitude = ""
            this.input_langitude = ""
        }
    },
    beforeDestroy() {
        if (this.map) {
            this.map.dispose();
        }
    }
};
</script>
  
<style scoped>
#map {
    width: 100%;
    height: 1000px;
}
</style>
  
  