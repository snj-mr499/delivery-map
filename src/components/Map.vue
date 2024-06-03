<template>
    <v-card class="map-card">
        <v-card-title>Customer Locations</v-card-title>
        <v-card-text class="map-container">
            <l-map :zoom="zoom" :center="center" ref="mapContainer">
                <l-tile-layer :url="url" :attribution="attribution"></l-tile-layer>
                <l-marker v-for="(marker, index) in markers" :key="index" :lat-lng="marker.coordinates"></l-marker>
            </l-map>
        </v-card-text>
    </v-card>
</template>

<script>
import { ref, watch } from 'vue';
import { LMap, LTileLayer, LMarker } from 'vue3-leaflet';
import 'leaflet/dist/leaflet.css';

export default {
    props: ['addresses'],
    components: {
        LMap,
        LTileLayer,
        LMarker,
    },
    setup(props) {
        const mapContainer = ref(null);
        const zoom = ref(2);
        const center = ref([0, 0]);
        const url = ref('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png');
        const attribution = ref('&copy; <a href="https://www.openstreetmap.org/copyright">OpenStreetMap</a> contributors');
        const markers = ref([]);

        const geocodeAddress = async (address) => {
            try {
                const response = await fetch(`https://nominatim.openstreetmap.org/search?format=json&q=${address}`);
                const data = await response.json();
                console.log('Geocode response:', data); // Log the geocode response
                if (data && data.length > 0) {
                    return [parseFloat(data[0].lat), parseFloat(data[0].lon)];
                } else {
                    return null;
                }
            } catch (error) {
                console.error('Error geocoding address:', error);
                return null;
            }
        };

        const updateMarkers = async (newAddresses) => {
            for (let address of newAddresses) {
                const coordinates = await geocodeAddress(address);
                if (coordinates) {
                    markers.value.push({ address, coordinates });
                    center.value = coordinates; // Center the map to the new marker
                    zoom.value = 15; // Adjust the zoom level as needed
                } else {
                    console.warn(`Could not geocode address: ${address}`);
                }
            }
        };

        watch(() => props.addresses, async (newAddresses) => {
            await updateMarkers(newAddresses);
        }, { deep: true });

        return {
            mapContainer,
            zoom,
            center,
            url,
            attribution,
            markers,
        };
    }
};
</script>

<style scoped>
.map-container {
    height: 500px;
    width: 100%;
    margin: 15px 0;
}

.map-card {
    flex: 1;
    display: flex;
    flex-direction: column;
}
</style>