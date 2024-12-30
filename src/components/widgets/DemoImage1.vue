<template>
    <div>
        <img :src="imageUrl" alt="Demo Image" />
    </div>
</template>

<script setup>
import { defineProps } from 'vue';

const props = defineProps({
    widgetId: {
        type: String,
        required: true
    },
    imageUrl: {
        type: String,
        default: '/pic01.png'
    }
});

</script>

<style scoped>
img {
    max-width: 100%;
    height: auto;
}
</style>