<template>
    <div class="flex py-6 px-3 bg-customOrange/70 rounded-md shadow-md cursor-pointer">
        <div class="flex flex-col flex-1">
            <h2 class="text-3xl">{{ city.city }}</h2>
            <h3>{{ city.state }}</h3>
        </div>

        <div class="flex flex-col gap-2">
            <p class="text-3xl self-end">
                {{ Math.round(city.weather.current.temp_c) }}&deg;
            </p>
            <div class="flex gap-2">
                <span class="text-xs">
                    Feels like
                    {{ Math.round(city.weather.current.feelslike_c) }}&deg;
                </span>
            </div>
        </div>
    </div>
</template>

<script setup>
defineProps({
    city: {
        type: Object,
        default: () => { },
    },
});
</script>