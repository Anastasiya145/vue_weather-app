<script setup lang="ts">
import { ref } from 'vue';

const location = ref('');
const temperature = ref(0);
const description = ref('');
const loading = ref(false);
const error = ref('');
const formattedSearchQuery = ref('');

const weatherSearch = () => {
  if (formattedSearchQuery.value.trim().length > 0) {
    loading.value = true;

    fetch(`http://api.weatherapi.com/v1/current.json?key=e6708361bf9847b4a76155748240511&q=${formattedSearchQuery.value}`)
      .then((response) => response.json())
      .then((data) => {
        loading.value = false;
        error.value = '';
        const { location: loc, current } = data;
        location.value = loc.name;
        temperature.value = current.temp_c;
        description.value = current.condition.text;

        formattedSearchQuery.value = '';
      })
      .catch((err) => {
        loading.value = false;
        error.value = err.message;

        resetSearchData();
      });
  }
};

const resetSearchData = () => {
  location.value = '';
  temperature.value = 0;
  description.value = '';
};
</script>

<template>
  <div class="weather">
    <div class="container">
      <h1 class="weather-title"> Check the weather now in your city</h1>
      <div class="weather-form">
        <input
          type="search"
          class="weather-form__input"
          placeholder="Enter city"
          v-model="formattedSearchQuery"
          @keydown.enter="weatherSearch"
        />
        <button
          class="weather-form__btn"
          @click="weatherSearch"
          :disabled="loading || !formattedSearchQuery.trim()"
        >
          <span v-if="loading" class="loader"></span>
          <span v-else>Search</span>
        </button>
      </div>

      <div class="weather-info" v-if="!error && location && temperature && description">
        <p class="weather-info__text">{{ location }}</p>
        <p class="weather-info__text small">{{ temperature }} °C</p>
        <p class="weather-info__text">{{ description }}</p>
      </div>

      <div class="weather-error" v-else-if="error">
        {{ error }}
      </div>
    </div>

    <div class="weather-bg">
      <img
        class="weather-bg__img"
        :class="{ 'is-active': !description }"
        src="./assets/spatial.jpg"
        alt="App background"
      />
      <img
        v-for="weather in ['spatial', 'sunny', 'cloudy', 'rainy', 'overcast']"
        :key="weather"
        class="weather-bg__img"
        :class="{ 'is-active': description.toLowerCase().includes(weather)}"
        :src="`./src/assets/${weather}.jpg`"
        :alt="weather"
      />
    </div>
  </div>
</template>
