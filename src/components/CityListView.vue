<template>
    <div v-for="city in savedCities" :key="city.id">
        <CityCard :city="city" @click="goToCityView(city)" />
    </div>

    <p v-if="savedCities.length === 0" class="text-weather-secondary font-semibold">
        No locations added. To start tracking a location, search in
        the field above.
    </p>
</template>

<script setup>
import axios from "axios";
import { ref } from "vue";
import { useRouter } from "vue-router";
import CityCard from "./CityCard.vue";
import apiConfig from "../../apiConfig.json";

const savedCities = ref([]);
const getCities = async () => {
    if (localStorage.getItem("savedCities")) {
        savedCities.value = JSON.parse(
            localStorage.getItem("savedCities")
        );

        const requests = [];
        savedCities.value.forEach((city) => {
            requests.push(
                axios.get(
                    `${apiConfig.WEATHER_URL}current.json?q=${city.city}&key=${apiConfig.WEATHER_API_KEY}`
                )
            );
        });

        const weatherData = await Promise.all(requests);

        await new Promise((res) => setTimeout(res, 1000));

        weatherData.forEach((value, index) => {
            savedCities.value[index].weather = value.data;
        });
    }
};
await getCities();

const router = useRouter();
const goToCityView = (city) => {
    router.push({
        name: "cityView",
        params: { state: city.state, city: city.city },
        query: {
            id: city.id,
            lat: city.coords.lat,
            lng: city.coords.lng,
            alreadySaved: true,
        },
    });
};
</script>