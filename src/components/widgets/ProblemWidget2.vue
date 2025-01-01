<template>
    <div class="widget">
        <h3>{{ itemId }}</h3>
        <p> <u>{{ timeStr }}</u> </p>
        <button @click="handleClick">Click Me</button>
    </div>
</template>

<script setup>
import { defineProps, ref, onBeforeUnmount, toRefs } from 'vue';

const props = defineProps({
    itemId: {
        type: String,
        required: true
    },
});

const handleClick = () => {
    alert('Button clicked!');
};

const { itemId } = toRefs(props);

onBeforeUnmount(() => {
    console.log(`In DemoBlank2 onBeforeUnmount for item ${itemId.value}`);
});

const timeStr = ref(new Date().toLocaleTimeString());
setInterval(() => {
    timeStr.value = new Date().toLocaleTimeString();
}, 1000);
</script>

<style scoped>
.widget {
    height: 100%;
    background-color: darkseagreen;
    display: flex;
    flex-direction: column; /* Arrange children in a column */
    justify-content: center;
    align-items: center;
    margin: 20px;
    padding: 10px;
}

button {
    /* padding: 10px 20px; */
    font-size: 16px;
}

h3 {
    margin: 0;
}
</style>