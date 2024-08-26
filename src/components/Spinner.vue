<template>
    <div class="spinner"></div>
</template>

<script setup>
// No additional logic needed for this basic spinner
</script>

<style scoped>
.spinner {
    border: 4px solid rgba(0, 0, 0, 0.1);
    border-left-color: orange;
    border-radius: 50%;
    width: 50px;
    height: 50px;
    animation: spin 1s linear infinite;
    position: absolute;
    top: 50%;
    left: 50%;
}

@keyframes spin {
    to {
        transform: rotate(360deg);
    }
}
</style>