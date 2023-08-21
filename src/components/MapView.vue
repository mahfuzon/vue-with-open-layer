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
import Translate from 'ol/interaction/Translate';
import Collection from "ol/Collection"
import { fromLonLat } from 'ol/proj';

const centerCoordinate = [110.3695, -7.7956];
const center = fromLonLat(centerCoordinate);

const view = new View({
    center: center,
    zoom: 10
});

const tileLayer = new TileLayer({ source: new OSM() })

const locations = [
    {
        name: 'Lokasi A',
        latitude: -7.7864,
        longitude: 110.3658,
    },
    {
        name: 'Lokasi B',
        latitude: -7.8015,
        longitude: 110.3672,
    },
    {
        name: 'Lokasi C',
        latitude: -7.7937,
        longitude: 110.3708,
    },
];

const style = new Style({
    image: new Icon({
        scale: .7,
        anchor: [0.5, 1],
        src: '//raw.githubusercontent.com/jonataswalker/map-utils/master/images/marker.png'
    })
});



const source = new VectorSource();

export default {
    name: 'MapView',
    data() {
        return {
            input_latitude: "",
            input_langitude: "",
            listFeature: []
        }
    },
    mounted() {
        this.initializeApp();
        this.addMarkers();
    },
    methods: {
        initializeApp() {
            const map = new Map({
                target: 'map',
                layers: [tileLayer],
                view: view
            });

            map.on('pointermove', function (e) {
                if (e.dragging) return;
                const hit = map.hasFeatureAtPixel(map.getEventPixel(e.originalEvent));
                map.getTargetElement().style.cursor = hit ? 'pointer' : '';
            });
            
            this.map = map;
        },
        addMarkers() {
            locations.forEach(location => {
                const point = new Point(fromLonLat([location.longitude, location.latitude]));
                const vectorLayer = new VectorLayer({
                    source: source
                });

                const feature = new Feature({
                    geometry: point,
                    name: location.name,
                    draggable: true
                });

                this.listFeature.push(feature)

                feature.setStyle(style);
                source.addFeature(feature);

                const translate = new Translate({
                    features: new Collection([feature])
                });

                this.map.addInteraction(translate);

                this.map.addLayer(vectorLayer);
            });


        },
        addLocation() {
            if (this.input_latitude != "" && this.input_langitude != "") {
                locations.push({
                    name: `Lokasi ${locations.length + 1}`,
                    latitude: this.input_latitude,
                    longitude: this.input_langitude

                })
            }

            this.addMarkers()

            this.input_latitude = ""
            this.input_langitude = ""
        },
        beforeDestroy() {
            if (this.map) {
                this.map.dispose();
            }
        },
        setClusterStyle(feature) {
            const size = feature.get('features').length;
            const style = new Style({
                image: new Icon({
                    src: 'https://openlayers.org/en/latest/examples/data/icon.png',
                }),
                text: new Text({
                    text: size.toString(),
                    fill: new Fill({
                        color: '#fff',
                    }),
                }),
            });
            return style;
        },
        setCluster() {
            const features = locations.map((location) => {
                return new Feature({
                    geometry: new Point(fromLonLat([location.longitude, location.latitude])),
                    name: location.name,
                });
            });

            const clusterSource = new Cluster({
                distance: 40, // Minimum distance between features to be considered part of the same cluster (in pixels)
                source: new VectorSource({
                    features: features,
                }),
            });

            const clusterLayer = new VectorLayer({
                source: clusterSource,
                style: this.setClusterStyle,
            });

            this.map.addLayer(clusterLayer);
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
  
  