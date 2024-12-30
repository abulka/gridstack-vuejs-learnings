<template>
    <div>
        <img :src="imageUrl"
            alt="Demo Image" />
    </div>
</template>

<script setup>
import { defineProps } from 'vue';
import { onBeforeUnmount, toRefs } from 'vue';

const props = defineProps({
    itemId: {
        type: String,
        required: true
    },
    imageUrl: {
        type: String,
        default: '/pic01.png'
    }
});
const { itemId } = toRefs(props);

onBeforeUnmount(() => {
    console.log(`In DemoImage onBeforeUnmount for item ${itemId.value}`);
});
</script>

<style scoped>
img {
    max-width: 100%;
    height: auto;
}
</style>