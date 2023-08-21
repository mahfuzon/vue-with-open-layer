<template>
    <div id="map" style="width: 100%; height: 400px;"></div>
</template>
  
<script>
import 'ol/ol.css';
import Map from 'ol/Map';
import View from 'ol/View';
import { Tile as TileLayer, Vector as VectorLayer } from 'ol/layer';
import { OSM, Vector as VectorSource } from 'ol/source';
import { Icon, Style, Text } from 'ol/style';
import Feature from 'ol/Feature';
import Point from 'ol/geom/Point';
import { Cluster } from 'ol/source';
import { fromLonLat } from 'ol/proj';
import Fill from 'ol/style/Fill';

export default {
    name: 'MapView2',
    mounted() {
        this.initMap();
    },
    methods: {
        initMap() {
            const center = fromLonLat([110.3695, -7.7956]);
            const view = new View({
                center: center,
                zoom: 10,
            });

            const map = new Map({
                target: 'map',
                layers: [
                    new TileLayer({
                        source: new OSM(),
                    }),
                ],
                view: view,
            });

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
                style: this.clusterStyleFunction,
            });

            map.addLayer(clusterLayer);

            map.on('click', (event) => {
                const features = map.getFeaturesAtPixel(event.pixel);
                if (features.length > 0) {
                    const names = features[0].values_.features.map((feature) => {
                        return feature.values_.name
                    });
                    alert('cluster yang anda pilih terdapat lokasi: ' + names.join(', '));
                }
            });
        },
        clusterStyleFunction(feature) {
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
    },
};
</script>
  
<style></style>
  