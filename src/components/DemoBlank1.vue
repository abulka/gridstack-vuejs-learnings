<template>
    <div class="widget">
        <h3>{{ widgetId }}</h3>
        <p> <u>{{ timeStr }}</u> </p>
        <button @click="handleClick">Click Me</button>
    </div>
</template>

<script setup>
import { ref } from 'vue';

defineProps({
    widgetId: String,
});

const handleClick = () => {
    alert('Button clicked!');
};

const timeStr = ref(new Date().toLocaleTimeString());
setInterval(() => {
    timeStr.value = new Date().toLocaleTimeString();
}, 1000);
</script>

<style scoped>
.widget {
    height: 100%;
    background-color: red;
    display: flex;
    flex-direction: column; /* Arrange children in a column */
    justify-content: center;
    align-items: center;
}

button {
    padding: 10px 20px;
    font-size: 16px;
}
</style>