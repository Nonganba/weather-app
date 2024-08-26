<template>
    <div>
        <Suspense>
            <AsyncCityView />
            <template #fallback>
                <Spinner />
            </template>
        </Suspense>
    </div>
</template>

<script setup>
import AsyncCityView from "../components/AsyncCityView.vue";
import Spinner from "@/components/Spinner.vue";

</script>