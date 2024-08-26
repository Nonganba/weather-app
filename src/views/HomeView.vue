<template>
  <main class="container text-white">
    <div class="pt-4 mb-8 relative rounded-md">
      <input type="text" v-model="searchQuery" @input="getSearchResults" placeholder="Search for a city or state"
        class="py-2 px-1 text-weather-secondary font-bold w-full bg-transparent border-b focus:border-weather-secondary focus:outline-none focus:shadow-[0px_1px_0_0_#004E71]" />
      <ul class="absolute rounded-md bg-customOrange text-white w-full shadow-2xl py-2 px-4 top-[66px]"
        v-if="mapboxSearchResults">
        <p class="py-2" v-if="searchError">
          Sorry, something went wrong, please try again.
        </p>
        <p class="py-2" v-if="!searchError && mapboxSearchResults.length === 0">
          No results match your query, try a different term.
        </p>
        <template v-else>
          <li v-for="searchResult in mapboxSearchResults" :key="searchResult.id"
            class="py-2 cursor-pointer hover:text-weather-secondary" @click="previewCity(searchResult)">
            {{ searchResult.place_name }}
          </li>
        </template>
      </ul>
    </div>
    <div class="flex flex-col gap-4">
      <Suspense>
        <CityListView />
        <template #fallback>
          <Spinner />
        </template>
      </Suspense>
    </div>
  </main>
</template>

<script setup>
import { ref } from "vue";
import axios from 'axios';
import { useRouter } from "vue-router";
import CityListView from "@/components/CityListView.vue";
import Spinner from "@/components/Spinner.vue";

const router = useRouter();
const previewCity = (searchResult) => {
  const [city, state] = searchResult.place_name.split(',');
  router.push({
    name: 'cityView',
    params: { state: state.replaceAll(" ", ""), city: city },
    query: {
      lat: searchResult.geometry.coordinates[1],
      lng: searchResult.geometry.coordinates[0],
      preview: true,
    }
  })
};

const mapboxAPIKey = "pk.eyJ1Ijoibm9uZ3BvayIsImEiOiJjbTA4ZnVjazgwOTNyMmlzYjcxZzQzZnY5In0.Ahhob-t08jyqWRuZllRoUg";
const searchQuery = ref("");
const queryTimeout = ref(null);
const mapboxSearchResults = ref(null);
const searchError = ref(null);

const getSearchResults = () => {
  clearTimeout(queryTimeout.value);
  queryTimeout.value = setTimeout(async () => {
    if (searchQuery.value !== "") {
      try {
        const result = await axios.get(
          `https://api.mapbox.com/geocoding/v5/mapbox.places/${searchQuery.value}.json?access_token=${import.meta.env.VUE_APP_MAPBOX_API_KEY}&types=place`
        );
        mapboxSearchResults.value = result.data.features;
      } catch {
        searchError.value = true;
      }

      return;
    }
    mapboxSearchResults.value = null;
  }, 300);
};

</script>
