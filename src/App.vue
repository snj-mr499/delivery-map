<template>
    <v-app>
        <v-container>
            <AddressForm @addAddress="addAddress" />
            <Map :addresses="addresses" />
        </v-container>
    </v-app>
</template>

<script>
import { ref } from 'vue';
import AddressForm from './components/AddressForm.vue';
import Map from './components/Map.vue';

export default {
    name: 'App',
    components: {
        AddressForm,
        Map,
    },
    setup() {
        const addresses = ref([]);

        const addAddress = (address) => {
            addresses.value.push(address);
        };

        return {
            addresses,
            addAddress
        }
    }
}
</script>
