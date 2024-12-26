<template>
    <div class="widget">
        <button @click="handleClick">Click Me</button>
    </div>
</template>

<script setup>
import { ref } from 'vue';

const handleClick = () => {
    alert('Button clicked!');
};
</script>

<style scoped>
.widget {
    width: 200px;
    height: 200px;
    background-color: red;
    display: flex;
    justify-content: center;
    align-items: center;
}

button {
    padding: 10px 20px;
    font-size: 16px;
}
</style>