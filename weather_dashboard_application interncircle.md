```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Live Weather & 5-Day Forecast</title>
  
  <!-- Tailwind CSS -->
  <script src="https://cdn.tailwindcss.com"></script>
  <!-- FontAwesome Icons -->
  <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
  <!-- Google Fonts -->
  <link rel="preconnect" href="https://fonts.googleapis.com">
  <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
  <link href="https://fonts.googleapis.com/css2?family=Inter:wght@300;400;500;600;700;800&display=swap" rel="stylesheet">

  <style>
    body { font-family: 'Inter', sans-serif; }
    .weather-gradient {
      background: linear-gradient(135deg, #0f172a 0%, #1e293b 50%, #0f172a 100%);
    }
    .glass-card {
      background: rgba(255, 255, 255, 0.05);
      backdrop-filter: blur(12px);
      border: 1px solid rgba(255, 255, 255, 0.1);
    }
    .glass-card-light {
      background: rgba(255, 255, 255, 0.08);
      backdrop-filter: blur(8px);
      border: 1px solid rgba(255, 255, 255, 0.15);
    }
  </style>
</head>
<body class="weather-gradient text-slate-100 min-h-screen flex flex-col justify-between p-4 sm:p-6 md:p-8">

  <!-- Main App Container -->
  <main class="max-w-4xl w-full mx-auto my-auto space-y-6">
    
    <!-- App Header & Search Bar -->
    <header class="space-y-4">
      <div class="flex items-center justify-between">
        <div class="flex items-center space-x-3">
          <div class="w-10 h-10 rounded-xl bg-sky-500/20 border border-sky-400/30 flex items-center justify-center text-sky-400">
            <i class="fa-solid fa-cloud-sun text-xl"></i>
          </div>
          <div>
            <h1 class="text-xl font-bold tracking-tight text-white">Atmosphere Weather</h1>
            <p class="text-xs text-slate-400">Live Forecast & Climate Conditions</p>
          </div>
        </div>
        <span class="text-xs font-semibold px-3 py-1 rounded-full bg-sky-500/10 text-sky-300 border border-sky-500/20">
          Open-Meteo API
        </span>
      </div>

      <!-- Search Input Form -->
      <form id="searchForm" class="relative flex items-center">
        <div class="relative w-full">
          <i class="fa-solid fa-magnifying-glass absolute left-4 top-1/2 -translate-y-1/2 text-slate-400 text-sm"></i>
          <input 
            type="text" 
            id="searchInput" 
            placeholder="Search for a city (e.g., London, Tokyo, New York)..." 
            class="w-full bg-slate-800/80 text-white placeholder-slate-400 text-sm rounded-xl pl-11 pr-24 py-3.5 border border-slate-700 focus:outline-none focus:border-sky-500 focus:ring-1 focus:ring-sky-500 transition shadow-lg"
            autocomplete="off"
          />
          <button 
            type="submit" 
            class="absolute right-2 top-1/2 -translate-y-1/2 bg-sky-500 hover:bg-sky-400 active:bg-sky-600 text-slate-900 font-semibold text-xs px-4 py-2 rounded-lg transition shadow-md"
          >
            Search
          </button>
        </div>
      </form>

      <!-- Recent Searches Chips -->
      <div id="recentSearches" class="flex flex-wrap items-center gap-2 text-xs text-slate-400 pt-1">
        <span class="text-slate-500">Popular:</span>
        <button type="button" class="recent-chip bg-slate-800 hover:bg-slate-700 text-slate-300 px-2.5 py-1 rounded-md border border-slate-700/60 transition" data-city="London">London</button>
        <button type="button" class="recent-chip bg-slate-800 hover:bg-slate-700 text-slate-300 px-2.5 py-1 rounded-md border border-slate-700/60 transition" data-city="Tokyo">Tokyo</button>
        <button type="button" class="recent-chip bg-slate-800 hover:bg-slate-700 text-slate-300 px-2.5 py-1 rounded-md border border-slate-700/60 transition" data-city="New York">New York</button>
        <button type="button" class="recent-chip bg-slate-800 hover:bg-slate-700 text-slate-300 px-2.5 py-1 rounded-md border border-slate-700/60 transition" data-city="Paris">Paris</button>
      </div>
    </header>

    <!-- Friendly Error Message State -->
    <div id="errorBanner" class="hidden glass-card border-rose-500/40 bg-rose-500/10 text-rose-200 p-4 rounded-xl flex items-start space-x-3">
      <i class="fa-solid fa-circle-exclamation text-rose-400 text-lg mt-0.5 shrink-0"></i>
      <div>
        <h2 id="errorTitle" class="text-sm font-semibold text-rose-300">City Not Found</h2>
        <p id="errorMessage" class="text-xs text-rose-200/90 mt-0.5">Please check your spelling and try searching again.</p>
      </div>
    </div>

    <!-- Loading Skeleton State -->
    <div id="loadingState" class="hidden glass-card rounded-2xl p-6 space-y-6 animate-pulse">
      <div class="flex justify-between items-center">
        <div class="space-y-2">
          <div class="h-6 w-40 bg-slate-700/60 rounded"></div>
          <div class="h-4 w-24 bg-slate-700/40 rounded"></div>
        </div>
        <div class="h-12 w-12 bg-slate-700/60 rounded-full"></div>
      </div>
      <div class="h-16 w-32 bg-slate-700/60 rounded"></div>
      <div class="grid grid-cols-2 sm:grid-cols-4 gap-3 pt-4 border-t border-slate-700/50">
        <div class="h-16 bg-slate-700/40 rounded-xl"></div>
        <div class="h-16 bg-slate-700/40 rounded-xl"></div>
        <div class="h-16 bg-slate-700/40 rounded-xl"></div>
        <div class="h-16 bg-slate-700/40 rounded-xl"></div>
      </div>
    </div>

    <!-- Weather Content Container -->
    <section id="weatherContainer" class="space-y-6">
      
      <!-- Current Conditions Card -->
      <div class="glass-card rounded-2xl p-6 sm:p-8 shadow-2xl relative overflow-hidden">
        <div class="flex flex-col sm:flex-row justify-between sm:items-start gap-4">
          <div>
            <div class="flex items-center gap-2">
              <h2 id="cityName" class="text-2xl sm:text-3xl font-extrabold text-white">London</h2>
              <span id="countryCode" class="text-xs font-semibold px-2 py-0.5 rounded bg-slate-700 text-slate-300">GB</span>
            </div>
            <p id="currentDate" class="text-xs text-slate-400 mt-1">Thursday, Oct 29, 2026</p>
          </div>
          <div class="flex items-center gap-3 bg-slate-800/50 p-3 rounded-xl border border-slate-700/50 self-start">
            <div id="weatherIcon" class="text-3xl text-sky-400">
              <i class="fa-solid fa-cloud-sun"></i>
            </div>
            <div>
              <p id="weatherCondition" class="text-sm font-semibold text-white">Partly Cloudy</p>
              <p class="text-xs text-slate-400">Current Weather</p>
            </div>
          </div>
        </div>

        <!-- Temperature Display -->
        <div class="my-6 flex items-baseline gap-2">
          <span id="currentTemp" class="text-6xl font-black text-white tracking-tight">18</span>
          <span class="text-2xl font-bold text-sky-400">°C</span>
        </div>

        <!-- Key Metrics Grid -->
        <div class="grid grid-cols-2 sm:grid-cols-4 gap-3 pt-6 border-t border-slate-700/50">
          <div class="glass-card-light p-3 rounded-xl">
            <p class="text-xs text-slate-400 flex items-center gap-1.5"><i class="fa-solid fa-temperature-arrow-up text-amber-400"></i> Max Temp</p>
            <p id="maxTemp" class="text-base font-bold text-slate-100 mt-1">21°C</p>
          </div>
          <div class="glass-card-light p-3 rounded-xl">
            <p class="text-xs text-slate-400 flex items-center gap-1.5"><i class="fa-solid fa-temperature-arrow-down text-sky-400"></i> Min Temp</p>
            <p id="minTemp" class="text-base font-bold text-slate-100 mt-1">12°C</p>
          </div>
          <div class="glass-card-light p-3 rounded-xl">
            <p class="text-xs text-slate-400 flex items-center gap-1.5"><i class="fa-solid fa-wind text-teal-400"></i> Wind Speed</p>
            <p id="windSpeed" class="text-base font-bold text-slate-100 mt-1">14 km/h</p>
          </div>
          <div class="glass-card-light p-3 rounded-xl">
            <p class="text-xs text-slate-400 flex items-center gap-1.5"><i class="fa-solid fa-droplet text-blue-400"></i> Humidity</p>
            <p id="humidity" class="text-base font-bold text-slate-100 mt-1">65%</p>
          </div>
        </div>
      </div>

      <!-- 5-Day Forecast Grid -->
      <div class="space-y-3">
        <h3 class="text-sm font-semibold text-slate-300 flex items-center gap-2">
          <i class="fa-solid fa-calendar-days text-sky-400"></i> 5-Day Forecast
        </h3>
        <div id="forecastGrid" class="grid grid-cols-2 sm:grid-cols-5 gap-3">
          <!-- Dynamic 5-day forecast cards inserted via JS -->
        </div>
      </div>

    </section>

  </main>

  <!-- Footer -->
  <footer class="text-center text-xs text-slate-500 py-4">
    Weather Dashboard &bull; Powered by Open-Meteo & Nominatim Geocoding API
  </footer>

  <!-- Application Logic -->
  <script>
    // --- API ENDPOINTS ---
    const API_CONFIG = {
      GEOCODING_URL: 'https://nominatim.openstreetmap.org/search',
      WEATHER_URL: 'https://api.open-meteo.com/v1/forecast'
    };

    // --- DOM ELEMENTS ---
    const searchForm = document.getElementById('searchForm');
    const searchInput = document.getElementById('searchInput');
    const weatherContainer = document.getElementById('weatherContainer');
    const loadingState = document.getElementById('loadingState');
    const errorBanner = document.getElementById('errorBanner');
    const errorTitle = document.getElementById('errorTitle');
    const errorMessage = document.getElementById('errorMessage');

    // Weather Display Elements
    const cityNameEl = document.getElementById('cityName');
    const countryCodeEl = document.getElementById('countryCode');
    const currentDateEl = document.getElementById('currentDate');
    const weatherConditionEl = document.getElementById('weatherCondition');
    const weatherIconEl = document.getElementById('weatherIcon');
    const currentTempEl = document.getElementById('currentTemp');
    const maxTempEl = document.getElementById('maxTemp');
    const minTempEl = document.getElementById('minTemp');
    const windSpeedEl = document.getElementById('windSpeed');
    const humidityEl = document.getElementById('humidity');
    const forecastGridEl = document.getElementById('forecastGrid');

    let debounceTimer = null;

    // --- WMO WEATHER CODE TRANSLATOR ---
    function getWeatherMeta(code) {
      if ([0].includes(code)) return { desc: 'Clear Sky', icon: 'fa-sun text-amber-400' };
      if ([1, 2].includes(code)) return { desc: 'Mainly Clear', icon: 'fa-cloud-sun text-amber-300' };
      if ([3].includes(code)) return { desc: 'Overcast', icon: 'fa-cloud text-slate-300' };
      if ([45, 48].includes(code)) return { desc: 'Foggy', icon: 'fa-smog text-slate-400' };
      if ([51, 53, 55, 61, 63, 65].includes(code)) return { desc: 'Rain', icon: 'fa-cloud-showers-heavy text-sky-400' };
      if ([71, 73, 75, 77, 85, 86].includes(code)) return { desc: 'Snowfall', icon: 'fa-snowflake text-indigo-300' };
      if ([95, 96, 99].includes(code)) return { desc: 'Thunderstorm', icon: 'fa-cloud-bolt text-purple-400' };
      return { desc: 'Cloudy', icon: 'fa-cloud-sun text-sky-300' };
    }

    // --- ERROR HANDLING & UI STATES ---
    function showError(title, message) {
      loadingState.classList.add('hidden');
      weatherContainer.classList.add('hidden');
      errorTitle.textContent = title;
      errorMessage.textContent = message;
      errorBanner.classList.remove('hidden');
    }

    function showLoading() {
      errorBanner.classList.add('hidden');
      weatherContainer.classList.add('hidden');
      loadingState.classList.remove('hidden');
    }

    function showWeather() {
      loadingState.classList.add('hidden');
      errorBanner.classList.add('hidden');
      weatherContainer.classList.remove('hidden');
    }

    // --- ASYNC FETCH WEATHER ---
    async function fetchWeather(city) {
      const sanitizedCity = city.trim();
      if (!sanitizedCity) {
        showError('Empty Search Query', 'Please enter a valid city name to check the weather.');
        return;
      }

      showLoading();

      try {
        // 1. Geocode City Name using OpenStreetMap Nominatim
        const geocodeUrl = `${API_CONFIG.GEOCODING_URL}?q=${encodeURIComponent(sanitizedCity)}&format=json&limit=1`;
        const geoResponse = await fetch(geocodeUrl, {
          headers: { 'Accept-Language': 'en' }
        });

        if (!geoResponse.ok) {
          throw new Error(`Geocoding failed with status: ${geoResponse.status}`);
        }

        const geoData = await geoResponse.json();

        if (!geoData || geoData.length === 0) {
          showError('City Not Found', `We couldn't find "${sanitizedCity}". Please check spelling and try again.`);
          return;
        }

        const location = geoData[0];
        const lat = location.lat;
        const lon = location.lon;
        
        // Extract location name components safely
        const displayNameParts = location.display_name.split(',');
        const cityName = displayNameParts[0].trim();
        const countryCode = displayNameParts[displayNameParts.length - 1].trim().slice(0, 3).toUpperCase();

        // 2. Fetch Weather Data from Open-Meteo
        const weatherUrl = `${API_CONFIG.WEATHER_URL}?latitude=${lat}&longitude=${lon}&current_weather=true&daily=weathercode,temperature_2m_max,temperature_2m_min,relative_humidity_2m_max,windspeed_10m_max&timezone=auto`;
        const weatherResponse = await fetch(weatherUrl);

        if (!weatherResponse.ok) {
          throw new Error(`Weather service returned HTTP ${weatherResponse.status}`);
        }

        const weatherData = await weatherResponse.json();
        renderWeatherUI(cityName, countryCode, weatherData);

      } catch (err) {
        console.error('Weather Search Error:', err);
        showError('Network / Service Error', 'Unable to retrieve weather data right now. Please check your internet connection.');
      }
    }

    // --- RENDER WEATHER UI ---
    function renderWeatherUI(city, country, data) {
      const current = data.current_weather;
      const daily = data.daily;

      // Current Conditions
      cityNameEl.textContent = city;
      countryCodeEl.textContent = country;
      currentDateEl.textContent = new Date().toLocaleDateString('en-US', {
        weekday: 'long', month: 'short', day: 'numeric', year: 'numeric'
      });

      const weatherMeta = getWeatherMeta(current.weathercode);
      weatherConditionEl.textContent = weatherMeta.desc;
      weatherIconEl.innerHTML = `<i class="fa-solid ${weatherMeta.icon}"></i>`;
      
      currentTempEl.textContent = Math.round(current.temperature);
      maxTempEl.textContent = `${Math.round(daily.temperature_2m_max[0])}°C`;
      minTempEl.textContent = `${Math.round(daily.temperature_2m_min[0])}°C`;
      windSpeedEl.textContent = `${current.windspeed} km/h`;
      humidityEl.textContent = `${daily.relative_humidity_2m_max[0] || 60}%`;

      // 5-Day Forecast Rendering
      forecastGridEl.innerHTML = '';
      for (let i = 0; i < 5; i++) {
        const dateStr = daily.time[i];
        const dateObj = new Date(dateStr + 'T00:00:00');
        const dayName = i === 0 ? 'Today' : dateObj.toLocaleDateString('en-US', { weekday: 'short' });
        const meta = getWeatherMeta(daily.weathercode[i]);
        const maxT = Math.round(daily.temperature_2m_max[i]);
        const minT = Math.round(daily.temperature_2m_min[i]);

        const forecastCard = document.createElement('div');
        forecastCard.className = 'glass-card-light p-3.5 rounded-xl text-center space-y-2 border border-slate-700/40 hover:border-sky-500/50 transition';
        forecastCard.innerHTML = `
          <p class="text-xs font-semibold text-slate-300">${dayName}</p>
          <div class="text-2xl my-1"><i class="fa-solid ${meta.icon}"></i></div>
          <p class="text-[11px] text-slate-400 line-clamp-1">${meta.desc}</p>
          <div class="text-xs font-bold text-white pt-1">
            <span>${maxT}°</span>
            <span class="text-slate-400 text-[10px] font-normal ml-1">${minT}°</span>
          </div>
        `;
        forecastGridEl.appendChild(forecastCard);
      }

      showWeather();
    }

    // --- EVENT LISTENERS ---
    searchForm.addEventListener('submit', (e) => {
      e.preventDefault();
      fetchWeather(searchInput.value);
    });

    // Debounce live typing to respect API limits
    searchInput.addEventListener('input', () => {
      clearTimeout(debounceTimer);
      if (searchInput.value.trim().length >= 3) {
        debounceTimer = setTimeout(() => {
          fetchWeather(searchInput.value);
        }, 600);
      }
    });

    // Popular City Chips Click
    document.querySelectorAll('.recent-chip').forEach(chip => {
      chip.addEventListener('click', () => {
        const city = chip.getAttribute('data-city');
        searchInput.value = city;
        fetchWeather(city);
      });
    });

    // Initial Load
    document.addEventListener('DOMContentLoaded', () => {
      fetchWeather('London');
    });
  </script>
</body>
</html>
```