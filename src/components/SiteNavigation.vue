<template>
    <header class="sticky top-0 bg-customOrange/60 shadow-lg z-50">
        <nav class="container flex sm:flex-row items-center gap-4 text-white py-6">
            <RouterLink :to="{ name: 'home' }">
                <div class="flex items-center gap-3">
                    <i class="fa-solid fa-sun text-2xl"></i>
                    <p class="text-2xl ">SkyCast</p>
                </div>
            </RouterLink>

            <div class="flex gap-3 flex-1 justify-end">
                <i class="fas fa-map-marker-alt text-xl hover:text-weather-secondary duration-150 cursor-pointer mx-2"
                    @click="detectCurrentLocation"></i>
                <i class="fa-solid fa-circle-info text-xl hover:text-weather-secondary duration-150 cursor-pointer mx-2"
                    @click="toggleModal"></i>
                <i class="fa-solid fa-plus text-xl hover:text-weather-secondary duration-150 cursor-pointer ml-2"
                    @click="addCity" v-if="route.query.preview"></i>
            </div>

            <BaseModal :modalActive="modalActive" @close-modal="toggleModal">
                <div class="text-black">
                    <h1 class="text-2xl mb-1">About:</h1>
                    <p class="mb-4">
                        The Local Weather allows you to track the current and
                        future weather of cities of your choosing.
                    </p>
                    <h2 class="text-2xl">How it works:</h2>
                    <ol class="list-decimal list-inside mb-4">
                        <li>
                            Search for your city by entering the name into the
                            search bar or click on the location icon to detect current location.
                        </li>
                        <li>
                            Select a city within the results, this will take
                            you to the current weather for your selection.
                        </li>
                        <li>
                            Track the city by clicking on the "+" icon in the
                            top right. This will save the city to view at a
                            later time on the home page.
                        </li>
                    </ol>

                    <h2 class="text-2xl">Removing a city</h2>
                    <p>
                        If you no longer wish to track a city, simply select
                        the city within the home page. At the bottom of the
                        page, there will be am option to delete the city.
                    </p>
                </div>
            </BaseModal>
        </nav>
    </header>
</template>

<script setup>
import { RouterLink, useRoute, useRouter } from "vue-router";
import { uid } from "uid";
import { ref } from "vue";
import BaseModal from "./BaseModal.vue";
import axios from "axios";
import apiConfig from "../../apiConfig.json";

const previewCurrentCity = (searchResult, latitude, longitude) => {
    const [city, state] = searchResult.place_name.split(',');
    router.push({
        name: 'cityView',
        params: { state: state.replaceAll(" ", ""), city: city },
        query: {
            lat: latitude,
            lng: longitude,
            currentLocation: true,
        }
    })
};

const detectCurrentLocation = () => {
    // Check if Geolocation is supported
    if (navigator.geolocation) {
        // Get the current position
        navigator.geolocation.getCurrentPosition(
            async (position) => {
                // Success callback
                const latitude = position.coords.latitude;
                const longitude = position.coords.longitude;
                const url = `${apiConfig.MAPBOX_URL}${longitude},${latitude}.json?access_token=${apiConfig.MAPBOX_API_KEY}`
                try {
                    const result = await axios.get(url);
                    previewCurrentCity(result.data.features[3], latitude, longitude);
                } catch (err) {
                    console.log(err);
                }
                // You can use these coordinates to display the location on a map or to find more information using a geolocation service.
            },
            (error) => {
                // Error callback
                switch (error.code) {
                    case error.PERMISSION_DENIED:
                        alert("User denied the request for Geolocation.");
                        break;
                    case error.POSITION_UNAVAILABLE:
                        alert("Location information is unavailable.");
                        break;
                    case error.TIMEOUT:
                        alert("The request to get user location timed out.");
                        break;
                    case error.UNKNOWN_ERROR:
                        alert("An unknown error occurred.");
                        break;
                }
            }
        );
    } else {
        alert("Geolocation is not supported by this browser.");
    }
};

const savedCities = ref([]);
const route = useRoute();
const router = useRouter();
const addCity = () => {
    if (localStorage.getItem("savedCities")) {
        savedCities.value = JSON.parse(
            localStorage.getItem("savedCities")
        );
    }

    const locationObj = {
        id: uid(),
        state: route.params.state,
        city: route.params.city,
        coords: {
            lat: route.query.lat,
            lng: route.query.lng,
        },
    };

    savedCities.value.push(locationObj);
    localStorage.setItem(
        "savedCities",
        JSON.stringify(savedCities.value)
    );

    let query = Object.assign({}, route.query);
    delete query.preview;
    query.id = locationObj.id;
    query.alreadySaved = true;
    router.replace({ query });
};

const modalActive = ref(null);
const toggleModal = () => {
    modalActive.value = !modalActive.value;
};

detectCurrentLocation();
</script>