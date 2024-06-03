<template>
    <v-form @submit.prevent="handleSubmit">
        <v-text-field v-model="address" label="Customer Address" required></v-text-field>
        <v-btn type="submit" color="primary">Add Address</v-btn>
    </v-form>
</template>

<script>
import { ref } from 'vue';

export default {
    setup(_, { emit }) {
        const address = ref('');

        const handleSubmit = () => {
            emit('addAddress', address.value);
            address.value = '';
        };

        return {
            address,
            handleSubmit
        };
    }
};
</script>

<style scoped>
.v-form {
    display: flex;
    align-items: center;
    gap: 10px;
    margin: 20px 0;
}
</style>