<script setup lang="ts">
const cookie = useCookie("city");
const config = useRuntimeConfig();

if (!cookie.value) cookie.value = "Pretoria";

const searchTerm = ref(cookie.value);
const input = ref("");
const background = ref("");

const {
  data: city,
  error,
  refresh,
} = useAsyncData(
  "city",
  async () => {
    let response;
    try {
      response = await $fetch(
        `https://api.openweathermap.org/data/2.5/weather/?q=${searchTerm.value}`,
        {
          params: {
            units: "metric",
            appid: config.public.WEATHER_APP_SECRET,
          },
        }
      );
      cookie.value = searchTerm.value;

      const temp = response.main.temp;

      if (temp <= -10) {
        background.value =
          "https://images.unsplash.com/photo-1483664852095-d6cc6870702d?ixlib=rb-1.2.1&ixid=MnwxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8&auto=format&fit=crop&w=3540&q=80";
      } else if (temp > -10 && temp <= 0) {
        background.value =
          "https://images.unsplash.com/photo-1476820865390-c52aeebb9891?ixlib=rb-1.2.1&ixid=MnwxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8&auto=format&fit=crop&w=3540&q=80";
      } else if (temp > 0 && temp <= 10) {
        background.value =
          "https://images.unsplash.com/photo-1560258018-c7db7645254e?ixlib=rb-1.2.1&ixid=MnwxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8&auto=format&fit=crop&w=4032&q=80";
      } else {
        background.value =
          "https://images.unsplash.com/photo-1507525428034-b723cf961d3e?ixlib=rb-1.2.1&ixid=MnwxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8&auto=format&fit=crop&w=3546&q=80";
      }
    } catch (e) {}
    return response;
  },
  {
    watch: [searchTerm],
  }
);

const handleClick = () => {
  const formattedSearch = input.value.trim().split(" ").join("+");
  searchTerm.value = formattedSearch;
  input.value = "";
};
const goBack = () => {
  searchTerm.value = cookie.value;
}
const today = new Date().toLocaleDateString('en-US',{
  weekday: 'long',
  day: 'numeric',
  month: 'long',
  year:'numeric'
})
</script>

<template>
  <div v-if="city" class="h-screen relative overflow-hidden">
    <img :src="background" />
    <div class="absolute w-full h-full top-0 overlay" />
    <div class="absolute w-full h-full top-0 p-48">
      <div class="flex justify-between">
        <div>
          <h1 class="text-7xl text-white">{{ city?.name }}</h1>
          <p class="font-extralight text-2xl m-2 text-white">{{today}}</p>
          <img
            :src="`https://openweathermap.org/img/wn/${city?.weather[0].icon}@4x.png`"
            class="w-56 icon"
          />
        </div>
        <div>
          <p class="text-9xl text-white font-extralight">
            {{ city?.main.temp }}°C
          </p>
        </div>
      </div>
      <div class="mt-20">
        <input
          type="text"
          class="w-1/2 h-10"
          placeholder="Cape Town"
          v-model="input"
        />
        <button class="bg-sky-400 w-20 text-white h-10" @click="handleClick">
          Search
        </button>
      </div>
    </div>
  </div>
  <div v-else class="p-10">
    <h1 class="text-7xl"> City not found :)</h1>
    <button class="bg-sky-400 w-20 text-white h-10 rounded mt-5" @click="goBack"> Go back</button>
  </div>
</template>

<style scoped>
.overlay {
  background-color: rgba(0, 0, 0, 0.5);
}
.icon {
  margin-left: -2.75rem;
  margin-top: -2.5rem;
}
</style>
