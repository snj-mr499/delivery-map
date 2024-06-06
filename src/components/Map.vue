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
import { ref, watch, onMounted, nextTick } from 'vue';
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
        const zoom = ref(6);
        const center = ref([13.736717, 100.523186]);
        const url = ref('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png');
        const attribution = ref('&copy; <a href="https://www.openstreetmap.org/copyright">OpenStreetMap</a> contributors');
        const markers = ref([]);
        let mapInstance = null

        const initializeMapInstance = () => {
            if (mapContainer.value && mapContainer.value.mapObject) {
                mapInstance = mapContainer.value.mapObject
            }
        };

        onMounted(() => {
            initializeMapInstance()
        });

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
            markers.value = []
            let lastCoordinates = null
            for (let address of newAddresses) {
                const coordinates = await geocodeAddress(address);
                if (coordinates) {
                    markers.value.push({ address, coordinates });
                    lastCoordinates = coordinates; // Center the map to the new marker
                    // zoom.value = 15; // Adjust the zoom level as needed
                } else {
                    console.warn(`Could not geocode address: ${address}`);
                }
            }
            if (lastCoordinates) {
                center.value = lastCoordinates
                zoom.value = 15
                await nextTick()
                if (mapInstance) {
                    mapInstance.setView(lastCoordinates, 15)
                } else {
                    console.warn('Map instance is not available yet')
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