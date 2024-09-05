<template>
  <div class="min-h-screen flex items-center justify-center bg-cover bg-center bg-no-repeat" style="background-image: url('https://media.gettyimages.com/id/1386378648/photo/green-rice-fields-in-thailand.jpg?s=612x612&w=0&k=20&c=eBsLUUmPQ9NvZkiHypEnlDA-G17np6wxXIKKb9haiRM=');">
    <div class="p-6 lg:p-12 rounded-xl flex flex-col lg:flex-row lg:w-3/4 space-y-6 lg:space-y-0 lg:space-x-8 shadow-lg">

      <!-- City Selection -->
      <div class="backdrop-blur-lg p-4 rounded-xl mb-6">
        <select v-model="selectedCity" @change="fetchWeatherData">
          <option value="">Select a City</option>
          <option v-for="(city, index) in cities" :key="index" :value="city">{{ city }}</option>
        </select>
      </div>

      <!-- Weather Summary -->
      <div class="bg-orange-200 p-6 rounded-xl flex flex-col items-center space-y-4 text-center">
        <div class="text-lg font-semibold">Current Weather</div>
        <div class="text-6xl font-bold text-orange-500">{{ temperature }}°</div>
        <div class="text-md font-medium text-gray-700">{{ condition }}</div>
        <div class="text-gray-500">{{ selectedCity }}</div>
        <div class="text-sm text-gray-600">{{ currentDate }}</div>
        <div class="text-sm text-gray-600">Feels like {{ feelsLike }}° | Sunset {{ sunset }}</div>
      </div>

      <!-- Forecast Data -->
      <div class="bg-transparent backdrop-blur-lg rounded-xl p-4 shadow-md flex-1 space-y-4 w-full h-40 ">
        <div v-if="!selectedCity" class="text-center text-gray-500">
          Select a city to view the weather forecast
        </div>

        <div class="grid grid-cols-3 gap-2" v-if="hours.length">
          <div v-for="(hour, index) in hours.slice(0, 3)" :key="index" class="text-sm flex flex-col items-center">
            <span class="text-xs text-gray-500">{{ getForecastTime(hour.time) }}</span>
            <div class="flex items-center mt-1">
              <span class="weather-icon mr-2">{{ getConditionIcon(hour.condition) }}</span>
              <span class="text-lg">{{ hour.temp }}°</span>
            </div>
          </div>
        </div>
        <div class="border-t border-white"></div>

        <div class="grid grid-cols-3 gap-2" v-if="hours.length">
          <div v-for="(hour, index) in hours.slice(3, 6)" :key="index" class="text-sm flex flex-col items-center">
            <span class="text-xs text-gray-500">{{ getForecastTime(hour.time) }}</span>
            <div class="flex items-center mt-1">
              <span class="weather-icon mr-2">{{ getConditionIcon(hour.condition) }}</span>
              <span class="text-lg">{{ hour.temp }}°</span>
            </div>
          </div>
        </div>

        <!-- Random Quote -->
        <div class="mt-8 text-black italic text-center">
          "{{ randomQuote }}"
        </div>
      </div>
    </div>
  </div>
</template>


<script lang="ts">
import { defineComponent } from 'vue';
import axios from 'axios';

export default defineComponent({
  data() {
    return {
      cities: ["Delhi","Moscow","Paris","Los Angeles", "New York City", "London"],
      selectedCity: "",
      hours: [],
      randomQuote: "",
      temperature: 0,
      condition: "",
      feelsLike: 0,
      sunset: ""
    };
  },

  computed: {
    currentDate(): string {
      return new Date().toLocaleDateString('en-US', { weekday: 'long', year: 'numeric', month: 'long', day: 'numeric' });
    },
    country(): string {
      const cityIndex = this.cities.indexOf(this.selectedCity);
      return this.city === this.selectedCity ? this.city : this.cities[(cityIndex + 1) % this.cities.length];
    }
  },

  methods: {
    async fetchWeatherData() {
      try {
        if (!this.selectedCity.trim()) {
          throw new Error('Please select a city');
        }

        const response = await axios.get(`https://api.openweathermap.org/data/2.5/weather?q=${this.selectedCity}&appid=b7ebbeab0ca7076c2706c4728b663a0b&units=metric`);
        const weatherData = response.data;
        
        this.temperature = Math.round(weatherData.main.temp);
        this.condition = weatherData.weather[0].description;
        this.feelsLike = Math.round(weatherData.main.feels_like);
        
        const forecastResponse = await axios.get(`https://api.openweathermap.org/data/2.5/forecast?q=${this.selectedCity}&appid=b7ebbeab0ca7076c2706c4728b663a0b&units=metric`);
        this.hours = forecastResponse.data.list.slice(0, 12).map((hour: any) => ({
          time: new Date(hour.dt * 1000).toLocaleTimeString('en-US', { hour: 'numeric', minute: '2-digit' }),
          temp: Math.round(hour.main.temp),
          condition: hour.weather[0].main
        }));
        
        this.randomQuote = "Keep your face always toward the sunshine—and shadows will fall behind you.";
        
        const sunsetTime = new Date(new Date().setHours(18, 20, 0));
        this.sunset = sunsetTime.toLocaleTimeString('en-US', { hour: 'numeric', minute: '2-digit', hour12: true });

      } catch (error) {
        console.error('Error fetching weather data:', error);
        this.$notify({
          title: 'Error',
          text: error.message || 'Failed to fetch weather data. Please check your input and try again.',
          type: 'danger'
        });
      }
    },
    
    getConditionIcon(condition: string): string {
      switch (condition) {
        case 'Rain':
          return '🌧️';
        case 'Clouds':
          return '☁️';
        case 'Clear':
          return '☀️';
        default:
          return '❓';
      }
    },

    getForecastTime(timeString: string): string {
      const timeParts = timeString.split(' ');
      const hour = parseInt(timeParts[0], 10);
      
      if (hour === 0) return "Midnight";
      if (hour === 12) return "Noon";
      
      if (hour > 12) {
        hour -= 12;
        return `${hour} PM`;
      }
      
      return `${hour} AM`;
    }
  },

  mounted() {
    this.fetchWeatherData();
  }
});
</script>
