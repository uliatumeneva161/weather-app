<script>
import sunny from '@/assets/img/sun.jpg';
import cloudy from '@/assets/img/cloud.jpg';
import rainy from '@/assets/img/rain.jpg';
import overcast from '@/assets/img/overcast.jpg';
import snowy from '@/assets/img/snow.jpg';
import windy from '@/assets/img/wind.jpg';
import clear from '@/assets/img/clear.jpg';
import defaultBg from '@/assets/img/bg2.jpg';

export default {
  data() {
    return {
      time: this.getCurrentTime(),
      location: '',
      temperature: null,
      feelsLike: null,
      description: '',
      wind: null,
      windGust: null,
      pressure: null,
      humidity: null,
      precip: null,
      uvIndex: null,
      visibility: null,
      cloudCover: null,
      wind_dir: '',
      forecast: [],
      isLoading: false,
      isError: false,
      searchQuery: '',
      suggestions: [],
      showSuggestions: false,
      weatherImages: {
        sunny,
        cloudy,
        rainy,
        overcast,
        snowy,
        windy,
        clear,
        default: defaultBg
      },
      weatherIcons: {
        'Sunny': 'wb_sunny',
        'Partly cloudy': 'partly_cloudy_day',
        'Cloudy': 'cloud',
        'Overcast': 'cloud_queue',
        'Rain': 'rainy',
        'Light rain': 'rainy_light',
        'Moderate rain': 'rainy',
        'Heavy rain': 'rainy_heavy',
        'Snow': 'ac_unit',
        'Light snow': 'weather_snowy',
        'Moderate snow': 'weather_snowy',
        'Heavy snow': 'weather_snowy_heavy',
        'Windy': 'air',
        'Clear': 'clear_day',
        'Mist': 'foggy',
        'Fog': 'foggy',
        'Thunderstorm': 'thunderstorm'
      },
      windDirections: {
        'N': 'Северный',
        'NNE': 'Северо-северо-восточный',
        'NE': 'Северо-восточный',
        'ENE': 'Восточно-северо-восточный',
        'E': 'Восточный',
        'ESE': 'Восточно-юго-восточный',
        'SE': 'Юго-восточный',
        'SSE': 'Юго-юго-восточный',
        'S': 'Южный',
        'SSW': 'Юго-юго-западный',
        'SW': 'Юго-западный',
        'WSW': 'Западно-юго-западный',
        'W': 'Западный',
        'WNW': 'Западно-северо-западный',
        'NW': 'Северо-западный',
        'NNW': 'Северо-северо-западный'
      },
      weatherTranslations: {
        'Sunny': 'Солнечно',
        'Partly cloudy': 'Частично облачно',
        'Cloudy': 'Облачно',
        'Overcast': 'Пасмурно',
        'Rain': 'Дождь',
        'Light rain': 'Небольшой дождь',
        'Moderate rain': 'Умеренный дождь',
        'Heavy rain': 'Сильный дождь',
        'Snow': 'Снег',
        'Light snow': 'Небольшой снег',
        'Moderate snow': 'Умеренный снег',
        'Heavy snow': 'Сильный снег',
        'Windy': 'Ветрено',
        'Clear': 'Ясно',
        'Mist': 'Туман',
        'Fog': 'Густой туман',
        'Thunderstorm': 'Гроза'
      },
    }
  },
  mounted() {
    Object.values(this.weatherImages).forEach(imgSrc => {
    const img = new Image();
    img.src = imgSrc;
    img.onload = this.handleImageLoad;
    img.onerror = this.handleImageLoad;
  });
    this.$nextTick(() => {
      const input = this.$el.querySelector('.weather-form__input');
      if (input) input.focus();
    });
  },
  watch: {
    searchQuery(newVal) {
      if (newVal.length > 2) {
        this.fetchSuggestions();
      } else {
        this.suggestions = [];
        this.showSuggestions = false;
      }
    }
  },
  computed: {
    weatherClass() {
      const conditions = {
        'sunny': 'sunny',
        'cloudy': 'cloudy',
        'rain': 'rainy',
        'overcast': 'overcast',
        'snow': 'snowy',
        'wind': 'windy',
        'clear': 'clear',
        'thunderstorm': 'rainy',
        'mist': 'overcast',
        'fog': 'overcast'
      };

      const lowerCaseDescription = this.description.toLowerCase();
      for (const condition in conditions) {
        if (lowerCaseDescription.includes(condition)) {
          return conditions[condition];
        }
      }
      return 'default';
    },
    weatherIcon() {
      return this.weatherIcons[this.description] || 'help_outline';
    },
    currentWeatherImage() {
      return this.weatherImages[this.weatherClass] || this.weatherImages.default;
    },
    translatedDescription() {
      if (!this.description) return '';
      try {
        const normalizedDesc = this.description.trim();
        return this.weatherTranslations[normalizedDesc] || this.description;
      } catch (e) {
        console.error('Ошибка перевода:', e);
        return this.description;
      }
    },
    translatedWindDirection() {
      return this.windDirections[this.wind_dir] || this.wind_dir;
    },
    uvIndexLevel() {
      if (this.uvIndex <= 2) return 'Низкий';
      if (this.uvIndex <= 5) return 'Умеренный';
      if (this.uvIndex <= 7) return 'Высокий';
      if (this.uvIndex <= 10) return 'Очень высокий';
      return 'Экстремальный';
    }
  },
  methods: {
    getCurrentTime(locationData) { 
      if (locationData) {
        const localTime = new Date(locationData.localtime);
        return localTime.toLocaleTimeString('ru-RU', { 
          hour: '2-digit', 
          minute: '2-digit',
          timeZone: locationData.tz_id 
        });
      } else {
        return new Date().toLocaleTimeString('ru-RU', { 
          hour: '2-digit', 
          minute: '2-digit' 
        });
      }
    },
    async fetchSuggestions() {
      const apiKey = 'bfd00212b8de4671a78161428251603';
      try {
        const response = await fetch(`http://api.weatherapi.com/v1/search.json?key=${apiKey}&q=${this.searchQuery}`);
        if (!response.ok) throw new Error('Ошибка загрузки подсказок');
        this.suggestions = await response.json();
        this.showSuggestions = this.suggestions.length > 0;
      } catch (error) {
        console.error('Ошибка при получении подсказок:', error);
        this.suggestions = [];
        this.showSuggestions = false;
      }
    },
    hideSuggestions() {
      setTimeout(() => {
        this.showSuggestions = false;
      }, 200);
    },
    selectSuggestion(suggestion) {
      this.searchQuery = `${suggestion.name}, ${suggestion.country}`;
      this.suggestions = [];
      this.showSuggestions = false;
      this.searchWeather();
    },
    searchWeather() {
      this.isLoading = true;
      this.isError = false;
      this.suggestions = [];
      this.showSuggestions = false;
      
      const apiKey = 'bfd00212b8de4671a78161428251603';
      const url = `http://api.weatherapi.com/v1/forecast.json?key=${apiKey}&q=${this.searchQuery}&days=3&aqi=no&alerts=no`;
      
      fetch(url)
        .then(response => {
          if (!response.ok) {
            throw new Error('Network response was not ok');
          }
          return response.json();
        })
        .then(data => {
          this.isLoading = false;
          this.location = `${data.location.name}, ${data.location.country}`;
          this.time = this.getCurrentTime(data.location);
          this.temperature = data.current.temp_c;
          this.feelsLike = data.current.feelslike_c;
          this.description = data.current.condition.text;
          this.wind = (data.current.wind_kph / 3.6).toFixed(1);
          this.windGust = (data.current.gust_kph / 3.6).toFixed(1);
          this.precip = data.current.precip_mm;
          this.pressure = Math.round(data.current.pressure_mb * 0.750062);
          this.humidity = data.current.humidity;
          this.uvIndex = data.current.uv;
          this.visibility = data.current.vis_km;
          this.cloudCover = data.current.cloud;
          this.wind_dir = data.current.wind_dir;
          this.forecast = data.forecast.forecastday;
          this.searchQuery = '';
        })
        .catch(error => {
          this.isLoading = false;
          this.isError = true;
          console.error(error);
        });
    },
    getWeatherImage(condition) {
      const conditions = {
        'Sunny': this.weatherImages.sunny,
        'Partly cloudy': this.weatherImages.cloudy,
        'Cloudy': this.weatherImages.cloudy,
        'Overcast': this.weatherImages.overcast,
        'Rain': this.weatherImages.rainy,
        'Snow': this.weatherImages.snowy,
        'Windy': this.weatherImages.windy,
        'Clear': this.weatherImages.clear
      };

      const lowerCaseCondition = condition.toLowerCase();
      for (const cond in conditions) {
        if (lowerCaseCondition.includes(cond.toLowerCase())) {
          return conditions[cond];
        }
      }
      return this.weatherImages.default;
    }
  }
}
</script>

<template>
  <div class="weather">
    <div class="sky">
      <div class="clouds">
        <div class="cloud cloud-1"></div>
        <div class="cloud cloud-2"></div>
        <div class="cloud cloud-3"></div>
        <div class="cloud cloud-4"></div>
      </div>
    </div>
    
    <div class="container">
      <!-- Форма поиска -->
      <div class="weather-form">
        <div class="search-container">
          <input 
            ref="searchInput"
            type="text" 
            v-model="searchQuery" 
            class="search-input" 
            placeholder="Введите город"
            @input="fetchSuggestions"
            @focus="showSuggestions = true"
            @blur="hideSuggestions"
            @keyup.enter="searchWeather"
          >
          <button @click="searchWeather" class="search-button">
            <i class="material-icons">search</i>
          </button>
          
          <ul class="suggestions" v-show="showSuggestions && suggestions.length">
            <li 
              v-for="suggestion in suggestions" 
              :key="suggestion.id"
              @mousedown="selectSuggestion(suggestion)"
            >
              {{ suggestion.name }}, {{ suggestion.country }}
            </li>
          </ul>
        </div>
      </div>

      <!-- Состояние загрузки -->
      <div v-if="isLoading" class="weather-load">
        <i class="material-icons loading-icon">refresh</i>
        <p>Загрузка данных...</p>
      </div>

      <!-- Ошибка -->
      <div v-if="isError" class="weather-error">
        <i class="material-icons error-icon">error</i>
        <p>Ошибка при загрузке данных</p>
        <button @click="searchWeather" class="retry-button">Повторить</button>
      </div>

      <!-- Основная информация о погоде -->
      <div v-if="!isLoading && !isError && location" class="weather-info">
        <div class="weather-card" :style="{ backgroundImage: `url(${currentWeatherImage})` }">
          <div class="weather-overlay"></div>
          
          <div class="weather-content">
            <!-- Шапка с локацией и временем -->
            <div class="weather-header">
              <h2 class="location">{{ location }}</h2>
              <p class="time">
                <i class="material-icons">access_time</i>
                {{ time }}
              </p>
            </div>

            <!-- Основные показатели -->
            <div class="weather-main">
              <div class="temperature-block">
                <span class="temperature">{{ Math.round(temperature) }}°C</span>
                <span class="feels-like">&emsp;Ощущается как {{ Math.round(feelsLike) }}°C</span>
              </div>
            </div>

            <!-- Детализированная информация -->
            <div class="weather-details">

              <!-- <div class="detail-item">
                <i class="material-icons">{{weatherIcon}}</i>
                <span>{{translatedDescription}}</span>
              </div> -->

              <div class="detail-item">
                <i class="material-icons">air</i>
                <span>Ветер {{ wind }} м/с, {{ translatedWindDirection }}</span>
              </div>

              <div class="detail-item">
                <i class="material-icons">speed</i>
                <span>Давление {{ pressure }} мм рт.ст.</span>
              </div>

              <div class="detail-item">
                <i class="material-icons">water_drop</i>
                <span>Влажность {{ humidity }}%</span>
              </div>

              <div class="detail-item">
                <i class="material-icons">opacity</i>
                <span>Осадки {{ precip }} мм</span>
              </div>

              <div class="detail-item">
                <i class="material-icons">visibility</i>
                <span>Видимость {{ visibility }} км</span>
              </div>

              <div class="detail-item">
                <i class="material-icons">wb_sunny</i>
                <span>УФ-индекс {{ uvIndex }} ({{ uvIndexLevel }})</span>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
  .weather {
    width: 100%;
    min-height: 100vh;
    position: relative;
    overflow: hidden;
  }

  .sky {
    position: absolute;
    top: 0;
    left: 0;
    right: 0;
    bottom: 0;
    background: linear-gradient(135deg, #65bbfd 0%, #4292cb 100%);
    z-index: 0;
  }

  .clouds {
    position: absolute;
    inset: 0;
  }

  .cloud {
    position: absolute;
    background: white;
    border-radius: 50%;
    filter: blur(10px);
    opacity: 0.8;
    animation: moveClouds linear infinite;
  }

  .cloud:before,
  .cloud:after {
    content: '';
    position: absolute;
    background: white;
    border-radius: 50%;
  }

  .cloud-1 {
    width: 200px;
    height: 60px;
    top: 70%;
    left: -200px;
    animation-duration: 40s;
  }
  .cloud-1:before {
    width: 80px;
    height: 80px;
    top: -30px;
    left: 30px;
  }
  .cloud-1:after {
    width: 220px;
    height: 120px;
    top: -50px;
    left: 80px;
  }

  .cloud-2 {
    width: 300px;
    height: 100px;
    top: 30%;
    left: -300px;
    animation-duration: 60s;
    animation-delay: 10s;
  }
  .cloud-2:before {
    width: 100px;
    height: 100px;
    top: -30px;
    left: 50px;
  }
  .cloud-2:after {
    width: 150px;
    height: 150px;
    top: -60px;
    left: 150px;
  }

  .cloud-3 {
    width: 150px;
    height: 50px;
    top: 50%;
    left: -150px;
    animation-duration: 50s;
    animation-delay: 5s;
  }
  .cloud-3:before {
    width: 60px;
    height: 60px;
    top: -20px;
    left: 20px;
  }
  .cloud-3:after {
    width: 80px;
    height: 80px;
    top: -30px;
    left: 70px;
  }

  .cloud-4 {
    width: 250px;
    height: 80px;
    top: 65%;
    left: -250px;
    animation-duration: 70s;
    animation-delay: 15s;
  }
  .cloud-4:before {
    width: 90px;
    height: 90px;
    top: -30px;
    left: 40px;
  }
  .cloud-4:after {
    width: 120px;
    height: 120px;
    top: -50px;
    left: 130px;
  }

  .container {
    position: relative;
    z-index: 1;
    max-width: 1200px;
    margin: 0 auto;
    padding: 20px;
  }

  .weather-form {
    position: relative;
    max-width: 100%;
    margin: 0 auto 20px;
  }

  .search-container {
    width: 100%;
    position: relative;
    display: flex;
    align-items:center ;
    gap: 20px;
  }

  .search-input {
    flex: 1;
    padding: 10px 15px;
    font-size: 16px;
    border: 1px solid #ddd;
    border-radius: 10px;
    outline: none;
    height: 40px;
    transition: border-color 0.3s;
  }

  .search-input:focus {
    border-color: #4292cb;
  }
  .suggestions {
    position: absolute;
    top: 100%;
    left: 0;
    right: 0;
    background: white;
    border: 1px solid #ddd;
    border-radius: 0 0 4px 4px;
    max-height: 200px;
    overflow-y: auto;
    z-index: 10;
    margin: 0;
    padding: 0;
    list-style: none;
  }

  .suggestions li {
    padding: 10px 15px;
    cursor: pointer;
    transition: background-color 0.2s;
  }

  .suggestions li:hover {
    background-color: #f5f5f5;
  }

  .weather-load,
  .weather-error {
    text-align: center;
    padding: 40px;
    background: white;
    border-radius: 8px;
    max-width: 600px;
    margin: 40px auto;
  }

  .weather-info {
    margin-top: 20px;
  }

  .weather-card {
    position: relative;
    border-radius: 12px;
    overflow: hidden;
    color: white;
    min-height: 400px;
    background-size: cover;
    background-position: center;

  }

  .weather-overlay {
    position: absolute;
    top: 0;
    left: 0;
    right: 0;
    bottom: 0;
    background: rgba(0, 0, 0, 0.4);
  }

  .weather-content {
    position: relative;
    z-index: 1;
    padding: 25px;
  }

  .weather-header {
    margin-bottom: 20px;
  }

  .location {
    margin: 0;
    font-size: 28px;
  }

  .time {
    display: flex;
    align-items: center;
    margin: 5px 0 0;
    opacity: 0.9;
  }

  .temperature-block {
    margin: 30px 0;
  }

  .temperature {
    font-size: 48px;
    font-weight: bold;
  }

  .feels-like {
    opacity: 0.9;
  }

  .weather-details {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
    gap: 15px;
  }
  .detail-item {
    display: flex;
    align-items: center;
    gap: 10px;
    padding: 8px 0;
}

.detail-item.condition {
    background-color: blueviolet;
}

.material-icons {
    font-size: 20px;
}
  @keyframes moveClouds {
    to {
      transform: translateX(calc(100vw + 300px));
    }
  }

  @media (max-width: 768px) {
    .weather-form {
      max-width: 100%;
    }
    
    .weather-details {
      grid-template-columns: 1fr;
    }
    
    .temperature-block {
      flex-direction: column;
      align-items: flex-start;
      gap: 10px;
    }
  }
  </style>