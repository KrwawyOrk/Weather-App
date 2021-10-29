<template>
  <section class="w-full h-full">
    <div
      class="flex flex-col m-auto min-h-screen max-h-screen max-w-lg p-1 bg-cover bg-center"
      :class="weatherData.day === 'day' ? 'background-day' : 'background-night'"
    >
      <div class="px-2 py-1 rounded-md" style="background-color: #e46b1b">
        <h1 class="text-lg text-white font-bold">Weather App</h1>
      </div>

      <div class="flex flex-row justify-between mt-2">
        <input
          v-model="inputCity"
          class="
            w-8/12
            border
            bg-gray-100
            px-2
            py-1
            text-light
            rounded-l-md
            focus:outline-none
          "
          :placeholder="inputCity ? '' : 'Write city'"
        />
        <button
          @click="getWeatherData(inputCity)"
          class="
            bg-yellow-600
            hover:bg-yellow-700
            text-white
            font-bold
            py-1
            px-4
            rounded-r-md
            text-sm
            w-4/12
            focus:outline-none
          "
        >
          Get weather!
        </button>
      </div>
      <div class="flex justify-around px-2 py-1 mt-2">
        <button
          @click="getWeatherForRandomCity"
          class="
            bg-yellow-600
            hover:bg-yellow-700
            text-white
            font-bold
            py-1
            px-4
            rounded-md
            text-sm
            focus:outline-none
          "
        >
          Get random city!
        </button>
      </div>

      <section class="overflow-y-auto">
        <div
          v-if="cityFound"
          class="flex flex-col mt-4 space-y-4 min-w-full weather-animation"
        >
          <div class="border rounded-2xl shadow-xl px-10 py-10 mx-3 bg-white">
            <h1
              v-if="weatherData.city && weatherData.country"
              class="text-3xl font-bold text-center"
            >
              {{ weatherData.city }}, {{ weatherData.country }}
            </h1>
            <div class="flex flex-row space-x-2 mt-4 justify-center">
              <img
                v-if="weatherData.iconUrl"
                :src="weatherData.iconUrl"
                width="70"
                height="70"
              />
              <h1 v-if="weatherData.temperature" class="text-4xl font-bold">
                {{ weatherData.temperature }}<span>&#8451;</span>
              </h1>
            </div>
            <h1 v-if="adviceForTheDay" class="italic text-center text-sm mt-5">
              {{ adviceForTheDay }}
            </h1>
          </div>
          <div class="border rounded-2xl shadow-xl px-10 py-2 sm:py-10 mx-3 bg-white">
            <div class="font-light">
              <h1 v-if="weatherData.description">
                Description: {{ weatherData.description }}
              </h1>
              <h1 v-if="weatherData.temperatureFellsLike">
                Feels like: {{ weatherData.temperatureFellsLike
                }}<span>&#8451;</span>
              </h1>
              <h1 v-if="weatherData.temperatureMin">
                Temp min: {{ weatherData.temperatureMin }}<span>&#8451;</span>
              </h1>
              <h1 v-if="weatherData.temperatureMax">
                Temp max: {{ weatherData.temperatureMax }}<span>&#8451;</span>
              </h1>
            </div>
          </div>
          <div class="border rounded-2xl shadow-xl px-10 py-2 sm:py-2 mx-3 bg-white overflow-y-scroll max-h-32">
         
              <h1 v-for="city in lastCities" :key="city.city" class="font-light">
                {{ city.city }}, {{ city.country }} {{ city.temperature }}<span>&#8451;</span>
              </h1>
       
          </div>
        </div>
      </section>
    </div>
  </section>
</template>

<script>
import axios from "axios";
import { ref, onMounted } from "vue";

import { weather_api_key } from "../api_keys";
import { getRandomCapital } from "../functions/getRandomCapital";

export default {
  setup() {
    const inputCity = ref("");
    const cityFound = ref(false);
    const adviceForTheDay = ref("");
    const weatherApiKey = weather_api_key;
    const lastCities = ref([]);

    const weatherData = ref({
      city: "",
      country: "",
      temperature: "",
      temperatureFellsLike: "",
      temperatureMin: "",
      temperatureMax: "",
      description: "",
      iconUrl: "",
      day: "",
    });

    onMounted(() => {
      getWeatherData(getRandomCapital());
    });

    async function getWeatherData(city) {
      const weatherApiUrl = `https://api.openweathermap.org/data/2.5/weather?q=${city}&appid=${weatherApiKey}&units=metric`;
      const adviceApiUrl = "https://api.adviceslip.com/advice";

      await Promise.all([axios.get(weatherApiUrl), axios.get(adviceApiUrl)])
        .then((response) => {
          cityFound.value = false;
          inputCity.value = "Loading weather data...";

          //Weather
          const {
            data: { main, name, weather, sys },
          } = response[0];

          const ref = weatherData.value;
          ref.city = name;
          ref.country = sys.country;
          ref.temperature = parseInt(main.temp);
          ref.temperatureFellsLike = parseInt(main.feels_like);
          ref.temperatureMin = main.temp_min;
          ref.temperatureMax = main.temp_max;
          ref.description = weather[0].description;
          ref.iconUrl = `https://openweathermap.org/img/w/${weather[0].icon}.png`;
          ref.day = ref.iconUrl.includes("d.png") ? "day" : "night";

          //lastCities.value = [{city: ref.city, temperature: ref.temperature}, ...lastCities.value];
          lastCities.value.push({city: ref.city, country: ref.country, temperature: ref.temperature});
          console.log(lastCities.value);

          
          //Advice
          const {
            data: {
              slip: { advice },
            },
          } = response[1];

          adviceForTheDay.value = advice;
        })
        .finally(() => {
          inputCity.value = "";
          cityFound.value = true;
        });
    }

    function getWeatherForRandomCity() {
      getWeatherData(getRandomCapital());
    }

    return {
      cityFound,
      adviceForTheDay,
      inputCity,
      weatherData,
      lastCities,
      getWeatherData,
      getWeatherForRandomCity,
    };
  },
};
</script>

<style>
.weather-animation {
  animation-name: show_weather_data;
  animation-duration: 0.4s;
  animation-timing-function: ease-in-out;
}
@keyframes show_weather_data {
  0% {
    opacity: 0;
  }
  50% {
    opacity: 0.5;
  }
  100% {
    opacity: 1;   
  }
}

.background-day {
  background-image: url("../assets/morning.jpg");
}

.background-night {
  background-image: url("../assets/night.jpg");
}
</style>