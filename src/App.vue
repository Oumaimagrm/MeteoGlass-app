<template>
  
 <div 
  class="container"
  :style="{ backgroundImage: `url(${getBackgroundImage()})` }">


    <div class="card">
    <h1>MeteoGlass</h1>
    <p class="subtitle">Modern Weather Forecast</p>
    
    <input v-model="city" placeholder="Enter city" />
    <button @click="getWeather">Search</button>

    <div v-if="weather">
      <h2>{{ weather.name }}</h2>

        <!-- 🌤️ ICON -->
       <img :src="getIconUrl(weather.weather[0].icon)" />

      <p class="temp">{{ weather.main.temp }} °C</p>
      <p>{{ weather.weather[0].description }}</p>
    
    </div>
  </div>
  <h2 v-if="weather" class="section-title">
  Weather Details
</h2>

<div v-if="weather" class="details">

  <div class="detail-box">
    <p>💧</p>
    <span>Humidity</span>
    <h3>{{ weather.main.humidity }}%</h3>
  </div>

  <div class="detail-box">
    <p>🌬️</p>
    <span>Wind</span>
    <h3>{{ weather.wind.speed }} km/h</h3>
  </div>

  <div class="detail-box">
    <p>🌡️</p>
    <span>Feels Like</span>
    <h3>{{ Math.round(weather.main.feels_like) }}°</h3>
  </div>

  <div class="detail-box">
    <p>☁️</p>
    <span>Pressure</span>
    <h3>{{ weather.main.pressure }}</h3>
  </div>

</div>
  <h2 v-if="hourlyForecast.length" class="section-title">
  Hourly Weather
</h2>
  <div v-if="hourlyForecast.length" class="hourly-forecast">

  <div
    v-for="item in hourlyForecast"
    :key="item.dt"
    class="hour-card"
  >

    <p class="hour">
      {{
        new Date(item.dt_txt).toLocaleTimeString([], {
          hour: 'numeric'
        })
      }}
    </p>

    <img
      :src="getIconUrl(item.weather[0].icon)"
    />

    <p class="forecast-temp">
      {{ Math.round(item.main.temp) }}°
    </p>

  </div>

 </div>
 <h2 v-if="forecast.length" class="section-title">
  5-Day Forecast
</h2>
  <div v-if="forecast.length" class="forecast">
  <div
    v-for="item in forecast"
    :key="item.dt"
    class="forecast-card"
  >
    <p>
      {{
        new Date(item.dt_txt).toLocaleDateString('en-US', {
          weekday: 'short'
        })
      }}
    </p>

    <img
      :src="getIconUrl(item.weather[0].icon)"
    />

    <p class="forecast-temp">
  {{ Math.round(item.main.temp) }}°</p>
  </div>
</div>
</div>
 
</template>

<script setup>
import { ref } from 'vue'
import { onMounted, watch } from "vue"
import { Chart, registerables } from "chart.js"

const sunny = '/images/sunny.jpg'
const rainy = '/images/rainy.jpg'
const cloudy = '/images/cloudy.jpg'
const snow = '/images/snow.jpg'
const normal = '/images/default.jpg'

Chart.register(...registerables)

let chartInstance = null

const city = ref('')
const weather = ref(null)
const forecast = ref([])
const hourlyForecast = ref([])

const getIconUrl = (icon) => {
  return `https://openweathermap.org/img/wn/${icon}@2x.png`
}


const getWeather = async () => {
  const apiKey = 'f567747042c6d03fe68c91445ca151c3'

  const res = await fetch(
    `https://api.openweathermap.org/data/2.5/weather?q=${city.value}&appid=${apiKey}&units=metric`
  )

  const data = await res.json()
  weather.value = data

   // forecast
  const forecastRes = await fetch(
    `https://api.openweathermap.org/data/2.5/forecast?q=${city.value}&appid=${apiKey}&units=metric`
  )

  const forecastData = await forecastRes.json()

  forecast.value = forecastData.list.filter(item =>
    item.dt_txt.includes('12:00:00')
  )
  hourlyForecast.value = forecastData.list.slice(0, 8)
}

const getBackgroundImage = () => {
   if (!weather.value || !weather.value.weather) return normal

  const type = weather.value.weather[0].main

  switch (type) {
    case 'Clear':
      return sunny

    case 'Rain':
    case 'Drizzle':
    case 'Thunderstorm':
      return rainy

    case 'Clouds':
      return cloudy

    case 'Snow':
      return snow

    default:
      return sunny
  }
}
const renderChart = () => {
  const ctx = document.getElementById("hourChart")

  if (!ctx || !hourlyForecast.value.length) return

  if (chartInstance) chartInstance.destroy()

  chartInstance = new Chart(ctx, {
    type: "line",
    data: {
      labels: hourlyForecast.value.map((item) =>
        new Date(item.dt_txt).getHours() + ":00"
      ),
      datasets: [
        {
          label: "Temperature (°C)",
          data: hourlyForecast.value.map((item) =>
            Math.round(item.main.temp)
          ),
          borderColor: "#00c6ff",
          backgroundColor: "rgba(0,198,255,0.2)",
          tension: 0.4,
          fill: true,
        },
      ],
    },
    options: {
      responsive: true,
      plugins: {
        legend: { display: false },
      },
      scales: {
        x: {
          ticks: { color: "white" },
        },
        y: {
          ticks: { color: "white" },
        },
      },
    },
  })
}
watch(hourlyForecast, () => {
  renderChart()
})



</script>
 
<style>

h1 {
  font-size: 48px;
  margin-bottom: 25px;

  font-weight: 700;

  letter-spacing: 1px;

  text-shadow: 0 4px 10px rgba(0,0,0,0.3);
}
.temp {
  font-size: 55px;

  font-weight: bold;

  margin: 10px 0;
}
p {
  font-size: 18px;

  text-transform: capitalize;
}
body {
  margin: 0;
  padding: 0;
  font-family: Arial;
}
html, body, #app {
  width: 100%;
  height: 100%;
}

img {
  width: 70px;
  display: block;
  margin: 10px auto;

  animation: float 3s ease-in-out infinite;
}

@keyframes float {
  0% { transform: translateY(0); }

  50% { transform: translateY(-10px); }

  100% { transform: translateY(0); }
}
.forecast-temp {
  font-size: 22px;

  font-weight: bold;
}
.container {
  width: 100%;
  min-height: 100vh;

  display: flex;
  flex-direction: column;

  justify-content: center; /* مهم */
  align-items: center;

  gap: 25px;

  padding: 20px;

  background-size: cover;
  background-position: center;
  min-height: 100vh;
  overflow-x: hidden;
}
.card {
  width: 360px;

  max-width: 90%;

  padding: 30px;

  border-radius: 28px;

  background: rgba(255, 255, 255, 0.15);

  backdrop-filter: blur(15px);

  border: 1px solid rgba(255,255,255,0.2);

  box-shadow: 0 10px 40px rgba(0,0,0,0.25);

  text-align: center;

  color: white;

  animation: fadeIn 0.6s ease;
}

input {
  width: 100%;

  padding: 14px;

  border: none;

  outline: none;

  border-radius: 12px;

  margin-bottom: 15px;

  font-size: 16px;

  background: rgba(255,255,255,0.2);

  color: white;

  box-sizing: border-box;
}

input::placeholder {
  color: rgba(255,255,255,0.7);
}

button {
  width: 100%;

  padding: 14px;

  border: none;

  border-radius: 12px;

  background: rgba(255,255,255,0.25);

  color: white;

  font-size: 16px;

  font-weight: bold;

  cursor: pointer;

  transition: 0.3s;
}

button:hover {
  transform: scale(1.03);

  background: rgba(255,255,255,0.35);
}

button:hover {
  background: #00c6ff;
}

h2 {
  margin-top: 15px;
}
.forecast {
  width: 90%;
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(110px, 1fr));
  gap: 15px;

  padding: 10px;
}

.forecast-card {
  background: rgba(255,255,255,0.15);
  backdrop-filter: blur(12px);

  border-radius: 20px;

  padding: 15px;

  text-align: center;
  color: white;

  transition: 0.3s;

  animation: fadeIn 0.5s ease;
}
html, body {
  height: auto;
  min-height: 100%;
  overflow-x: hidden;
}
.forecast-card:hover {
  transform: translateY(-5px) scale(1.05);

  background: rgba(255,255,255,0.25);
}
@keyframes fadeIn {
  from {
    opacity: 0;
    transform: translateY(25px);
  }

  to {
    opacity: 1;
    transform: translateY(0);
  }
}

.forecast-card img {
  width: 50px;
}
.hourly-forecast {
  width: 90%;

  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(90px, 1fr));
  gap: 12px;

  padding: 10px;
}

.hour-card {
  background: rgba(255,255,255,0.15);
  backdrop-filter: blur(12px);

  border-radius: 18px;

  padding: 12px;

  text-align: center;
  color: white;

  transition: 0.3s;

  animation: fadeIn 0.5s ease;
}

.hour-card:hover {
  transform: translateY(-5px);
}

.hour {
  font-size: 14px;

  opacity: 0.8;
}
.section-title {
  width: 90%;

  color: white;

  font-size: 28px;
  font-weight: bold;

  margin-top: 10px;
  margin-bottom: -10px;

  text-align: left;

  text-shadow: 0 4px 10px rgba(0,0,0,0.3);
}
.details {
  width: 90%;

  display: grid;

  grid-template-columns: repeat(auto-fit, minmax(180px, 1fr));

  gap: 15px;
}

.details::-webkit-scrollbar {
  height: 6px;
}

.details::-webkit-scrollbar-thumb {
  background: rgba(255,255,255,0.3);

  border-radius: 10px;
}

.detail-box {
  background: rgba(255,255,255,0.12);

  padding: 15px;

  border-radius: 18px;

  backdrop-filter: blur(10px);

  text-align: center;

  transition: 0.3s;
}
@media (max-width: 768px) {

  .card {
    width: 95%;
  }

  .details {
    grid-template-columns: repeat(2, 1fr);
  }

}
.detail-box p,
.detail-box span,
.detail-box h3 {
  color: white;
}
.detail-box p {
  font-size: 28px;

  margin-bottom: 8px;
}
.subtitle {
  font-size: 14px;

  opacity: 0.8;

  margin-top: -15px;

  margin-bottom: 20px;

  letter-spacing: 1px;
}
/* 📱 MOBILE RESPONSIVE FIX */
@media (max-width: 768px) {

  .container {
    padding: 10px;
    justify-content: flex-start;
  }

  .card {
    width: 95%;
    padding: 20px;
    border-radius: 20px;
  }

  h1 {
    font-size: 32px;
  }

  .temp {
    font-size: 40px;
  }

  .details {
    grid-template-columns: repeat(2, 1fr);
    gap: 10px;
  }

  .detail-box {
    padding: 10px;
  }

  .forecast {
    grid-template-columns: repeat(2, 1fr);
  }

  .hourly-forecast {
    grid-template-columns: repeat(3, 1fr);
    overflow-x: hidden;
  }

  img {
    width: 50px;
  }
}
</style>