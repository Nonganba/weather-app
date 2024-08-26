<template>
    <div class="flex flex-col flex-1 items-center justify-center md:mx-20">
        <!-- Banner -->
        <div v-if="route.query.preview" class="text-white p-4 bg-weather-secondary/30 w-full text-center w-screen">
            <p>
                You are currently previewing this city, click the "+"
                icon to start tracking this city.
            </p>
        </div>
        <!-- Weather Overview -->
        <div
            class="flex flex-col md:flex-row md:gap-8 max-w-screen-md w-full md:items-start items-center justify-center">
            <div
                class="flex flex-col gap-4 flex-1 p-6 rounded-3xl shadow-lg text-center max-w-xs bg-customYellow/90 text-customOrange mt-4 min-w-[360px]">
                <div class="text-2xl font-medium text-customOrange">Today
                    <i class="fas fa-angle-down ml-3"></i>
                </div>
                <div class="text-7xl font-bold text-customOrange my-3 flex gap-4 items-center justify-center space-x-2">
                    <img class="w-[60px] h-auto" :src="`${weatherData.current.condition.icon}`
                        " alt="" />
                    <span>{{ Math.round(weatherData.current.temp_c) }}&deg;</span>
                </div>

                <div class="text-xl font-medium text-customOrange">
                    <p class="capitalize">
                        {{ weatherData.current.condition.text }}
                    </p>
                </div>
                <div class="text-customOrange">
                    {{ route.params.city }} , {{ route.params.state }}
                </div>
                <div class="text-customOrange">{{
                    new Date(weatherData.location.localtime).toLocaleDateString(
                        "en-IN",
                        {
                            weekday: "short",
                            day: "2-digit",
                            month: "long",
                        }
                    )
                }}
                </div>

                <div class="text-customOrange">
                    Feels like {{ Math.round(weatherData.current.feelslike_c) }} &deg; <span class="mx-2">|</span>
                    Sunset {{
                        weatherData.forecast.forecastday[0].astro.sunset }}
                </div>
            </div>

            <!-- Hourly Weather -->
            <div class="flex flex-col flex-1 items-center justify-center px-5 md:px-0">
                <div class="bg-customYellow/30 backdrop-blur-lg p-5 rounded-3xl shadow-lg mt-4 w-[360px]">
                    <div class=" text-white">
                        <div class="flex flex-row items-center justify-between text-center text-white">
                            <div v-for="hourData in weatherData.forecast.forecastday[0].hour.slice(0, 5)"
                                :key="hourData.time" class="flex flex-col">
                                <div class="text-sm uppercase">{{
                                    new Date(
                                        hourData.time
                                    ).toLocaleTimeString("en-IN", {
                                        hour: "numeric",
                                    })
                                }}</div>
                                <div class="text-sm flex flex-row items-center justify-center gap-1 mt-2">
                                    <img class="size-5" :src="`${hourData.condition.icon}`
                                        " alt="" />
                                    {{ Math.round(hourData.temp_c) }}&deg;
                                </div>
                            </div>
                        </div>

                        <hr class="my-4 border-gray-300" />

                        <div class="flex flex-row items-center justify-between text-center text-white">
                            <div v-for="hourData in weatherData.forecast.forecastday[0].hour.slice(5, 10)"
                                :key="hourData.time" class="flex flex-col">
                                <div class="text-sm uppercase">{{
                                    new Date(
                                        hourData.time
                                    ).toLocaleTimeString("en-IN", {
                                        hour: "numeric",
                                    })
                                }}</div>
                                <div class="text-sm flex flex-row items-center justify-center gap-1 mt-2">
                                    <img class="size-5" :src="`${hourData.condition.icon}`
                                        " alt="" />
                                    {{ Math.round(hourData.temp_c) }}&deg;
                                </div>
                            </div>
                        </div>
                    </div>
                </div>

                <div class="p-4 mt-1 text-white">
                    <h3 class="text-xl mb-2">Random Text</h3>
                    Lorem Ipsum is simply dummy text of the printing and typesetting industry. Lorem Ipsum has been the
                    industry's standard dummy text ever since the 1500s.
                </div>
            </div>
        </div>

        <!-- Weekly Weather -->
        <div
            class="max-w-screen-md w-[90%] py-6 bg-customYellow/30 backdrop-blur-lg rounded-3xl shadow-lg mt-4 mx-4 mb-4">
            <div class="mx-8 text-white">
                <h2 class="text-xl font-semibold text-center">7 Day Forecast</h2>
                <div v-for="day in sevenDayForecastData.forecast.forecastday" :key="day.date" class="flex items-center">
                    <p class="flex-1">
                        {{
                            new Date(day.date).toLocaleDateString(
                                "en-IN",
                                {
                                    day: 'numeric',
                                    month: 'short',
                                    weekday: "short",
                                }
                            )
                        }}
                    </p>
                    <img class="w-[50px] h-[50px] object-cover" :src="`${day.day.condition.icon}`
                        " alt="" />
                    <div class="flex gap-2 flex-1 justify-end">
                        <p>{{ Math.round(day.day.maxtemp_c) }}&deg</p>
                        <p>/</p>
                        <p>{{ Math.round(day.day.mintemp_c) }}&deg</p>
                    </div>
                </div>
            </div>
        </div>

        <div class="flex items-center gap-2 py-6 text-white cursor-pointer duration-150 hover:text-red-500"
            @click="removeCity" v-if="route.query.alreadySaved">
            <i class="fa-solid fa-trash"></i>
            <p>Remove City</p>
        </div>
    </div>
</template>

<script setup>
import axios from 'axios';
import { useRoute, useRouter } from 'vue-router';
import apiConfig from "../../apiConfig.json";

const route = useRoute();
const router = useRouter();
const getWeatherData = async () => {
    try {
        const weatherData = await axios.get(`${apiConfig.WEATHER_URL}forecast.json?q=${route.params.city}&days=1&key=${apiConfig.WEATHER_API_KEY}`);
        await new Promise((res) => setTimeout(res, 500));
        return weatherData.data;
    } catch (err) {
        alert('No data found for this location');
        router.push({
            name: "home",
        });
        console.log(err);
    }
};

const getSevenDayForecastData = async () => {
    try {
        const sevenDayForecastData = await axios.get(`${apiConfig.WEATHER_URL}forecast.json?q=${route.params.city}&days=7&tp=24&key=${apiConfig.WEATHER_API_KEY}`);
        await new Promise((res) => setTimeout(res, 500));
        return sevenDayForecastData.data;
    } catch (err) {
        console.log(err);
    }
};

const weatherData = await getWeatherData();
const sevenDayForecastData = await getSevenDayForecastData();

const removeCity = () => {
    const cities = JSON.parse(localStorage.getItem("savedCities"));
    const updatedCities = cities.filter(
        (city) => city.id !== route.query.id
    );
    localStorage.setItem(
        "savedCities",
        JSON.stringify(updatedCities)
    );
    router.push({
        name: "home",
    });
};

</script>