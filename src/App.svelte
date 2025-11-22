<script>
  import { onMount } from "svelte";

  // Global variables
  const API_KEY = process.env.API_KEY;
  let search_input = "";
  let loading = false;
  let currentData = null;
  let forecastData = null;

  // Get user location from ip
  onMount(async () => {
    loading = true;
    const ipData = await (await fetch("https://ipwho.is")).json();
    [currentData, forecastData] = await fetchData(
      `${ipData.latitude}, ${ipData.longitude}`
    );
    loading = false;
  });

  // async function to fetch weather data
  async function fetchData(cityOrCoordinates) {
    let url;
    // Fetch current data
    url = `https://api.weatherapi.com/v1/current.json?key=${API_KEY}&q=${cityOrCoordinates}`;
    const currentData = await (await fetch(url)).json();

    // Fetch forecast data
    url = `https://api.weatherapi.com/v1/forecast.json?key=${API_KEY}&q=${cityOrCoordinates}&days=14`;
    const forecastData = await (await fetch(url)).json();

    // Return data
    return [currentData, forecastData];
  }

  async function handleSubmit(e) {
		e.preventDefault()
    loading = true;
    // Fetch data and store it in variables
    [currentData, forecastData] = await fetchData(search_input.trim());
    loading = false;
  }
</script>

<main>
  <div id="main">
    <form method="post" on:submit={(e)=>handleSubmit(e)}>
      <div id="search-box">
        <input
          type="text"
          name="search"
          id="search-input"
          placeholder="Type Here"
          bind:value={search_input}
        />
        <button type="submit" id="search-btn"
          >Search</button
        >
      </div>
    </form>

    <div id="result-box">
      {#if loading}
        <div id="loader-container">
          <svg id="loader" viewBox="0 0 16 16">
            <circle
              cx="8"
              cy="8"
              r="7"
              stroke="currentColor"
              stroke-opacity="0.25"
              stroke-width="2"
              vector-effect="non-scaling-stroke"
              fill="none"
            ></circle>
            <path
              d="M15 8a7.002 7.002 0 00-7-7"
              stroke="currentColor"
              stroke-width="2"
              stroke-linecap="round"
              vector-effect="non-scaling-stroke"
              fill="none"
            ></path>
          </svg>
        </div>
      {:else if currentData !== null && forecastData !== null}
        {#if currentData.error}
          {#if currentData.error.code === 1006 || currentData.error.code === 1005}
            <p>No matching location found</p>
          {:else if currentData.error.code === 1003}
            <p>Please provide a city name or location</p>
          {:else}
            <p>Some error occured...</p>
          {/if}
        {:else}
          <div class="weather-layout">
            <!-- Current Weather Section -->
            <section class="current-weather card">
              <h2>Current Weather</h2>
              <img src={currentData.current.condition.icon} alt="Icon" />
              <p>
                <strong>Condition:</strong>
                {currentData.current.condition.text}
              </p>
              <p>
                <strong>City:</strong>
                {currentData.location.name}
              </p>
              <p>
                <strong>Region:</strong>
                {currentData.location.region}
              </p>
              <p>
                <strong>Country:</strong>
                {currentData.location.country}
              </p>
              <p>
                <strong>Temperature:</strong>
                {currentData.current.temp_c} °C
              </p>
              <p>
                <strong>Feels Like:</strong>
                {currentData.current.feelslike_c} °C
              </p>
              <p>
                <strong>Humidity:</strong>
                {currentData.current.humidity}%
              </p>
              <p>
                <strong>Wind:</strong>
                {currentData.current.wind_kph} kph
              </p>
              <p>
                <strong>UV Index:</strong>
                {currentData.current.uv}
              </p>
            </section>

            <!-- Forecast Section -->
            <section class="forecast-section card">
              <h2>Forecast</h2>
              <div class="forecast-grid">
                {#each forecastData.forecast.forecastday as day}
                  <div class="forecast-card card">
                    <h4>{day.date}</h4>
                    <img src={day.day.condition.icon} alt="Icon" />
                    <p>{day.day.condition.text}</p>
                    <p>
                      <strong>Max:</strong>
                      {day.day.maxtemp_c} °C
                    </p>
                    <p>
                      <strong>Min:</strong>
                      {day.day.mintemp_c} °C
                    </p>
                    <p>
                      <strong>Humidity:</strong>
                      {day.day.avghumidity}%
                    </p>
                    <p>
                      <strong>Wind:</strong>
                      {day.day.maxwind_kph} kph
                    </p>
                  </div>
                {/each}
              </div>
            </section>
          </div>
        {/if}
      {/if}
    </div>
  </div>
</main>

<style>
  * {
    margin: 0;
    padding: 0;
    box-sizing: border-box;
  }

  main {
    min-height: 100vh;
    background: linear-gradient(135deg, #0f0f23 0%, #1a1a2e 100%);
    padding: 2rem;
  }

  #main {
    max-width: 1400px;
    margin: 0 auto;
  }

  /* loader */
  #loader-container {
    display: flex;
    width: 100%;
    align-items: center;
    justify-content: center;
  }

  #loader {
    color: white;
    width: 100px;
    animation: spin 1s linear infinite;
  }

  @keyframes spin {
    0% {
      transform: rotate(0deg);
    }
    100% {
      transform: rotate(360deg);
    }
  }

  /* Search Section */
  #search-box {
    display: flex;
    gap: 1rem;
    margin-bottom: 2rem;
    justify-content: center;
    flex-wrap: wrap;
  }

  #search-input {
    flex: 1;
    min-width: 200px;
    max-width: 400px;
    padding: 0.875rem 1.25rem;
    background: rgba(255, 255, 255, 0.05);
    border: 1px solid rgba(255, 255, 255, 0.1);
    border-radius: 12px;
    color: #e4e4e7;
    font-size: 1rem;
    backdrop-filter: blur(10px);
  }

  #search-input::placeholder {
    color: rgba(228, 228, 231, 0.5);
  }

  #search-input:focus {
    outline: none;
    border-color: #3b82f6;
    background: rgba(255, 255, 255, 0.08);
    box-shadow: 0 0 0 3px rgba(59, 130, 246, 0.1);
  }

  #search-btn {
    padding: 0.875rem 1.5rem;
    background: linear-gradient(135deg, #3b82f6 0%, #1d4ed8 100%);
    color: white;
    border: none;
    border-radius: 12px;
    cursor: pointer;
    font-weight: 500;
    font-size: 0.95rem;
    transition: all 0.3s ease;
    box-shadow: 0 4px 15px rgba(59, 130, 246, 0.2);
  }

  #search-btn:hover {
    background: linear-gradient(135deg, #2563eb 0%, #1e40af 100%);
    transform: translateY(-2px);
    box-shadow: 0 6px 20px rgba(59, 130, 246, 0.3);
  }

  #search-btn:active {
    transform: translateY(0);
  }

  /* Layout for weather data */
  .weather-layout {
    display: grid;
    grid-template-columns: 1fr 1fr;
    gap: 2rem;
    margin-top: 2rem;
  }

  .card {
    background: rgba(255, 255, 255, 0.03);
    border: 1px solid rgba(255, 255, 255, 0.08);
    border-radius: 16px;
    padding: 1.5rem;
    backdrop-filter: blur(10px);
    transition: all 0.3s ease;
  }

  .card:hover {
    background: rgba(255, 255, 255, 0.05);
    border-color: rgba(255, 255, 255, 0.12);
    transform: translateY(-2px);
  }

  /* Current Weather */
  .current-weather {
    display: flex;
    flex-direction: column;
    align-items: center;
    gap: 0.75rem;
  }

  .current-weather h2 {
    font-size: 1.5rem;
    font-weight: 600;
    color: #f8fafc;
    text-align: center;
  }

  .current-weather img {
    width: 80px;
    height: 80px;
  }

  .current-weather p {
    display: flex;
    width: 100%;
    justify-content: space-between;
    font-size: 0.95rem;
    color: #cbd5e1;
  }

  /* Forecast Section */
  .forecast-section h2 {
    font-size: 1.5rem;
    font-weight: 600;
    color: #f8fafc;
    text-align: center;
    margin-bottom: 1rem;
  }

  .forecast-grid {
    display: flex;
    flex-direction: column;
    gap: 1rem;
    overflow-y: auto;
    max-height: 70vh;
  }

  .forecast-card {
    display: flex;
    justify-content: space-between;
    align-items: center;
    gap: 1rem;
    padding: 1rem;
    background: rgba(255, 255, 255, 0.03);
    border: 1px solid rgba(255, 255, 255, 0.08);
    border-radius: 12px;
  }

  .forecast-card h4 {
    font-size: 0.9rem;
    color: #94a3b8;
    font-weight: 500;
    min-width: 80px;
  }

  .forecast-card img {
    width: 40px;
    height: 40px;
  }

  .forecast-card p {
    font-size: 0.85rem;
    color: #e2e8f0;
  }

  /* Error Message */
  #result-box p {
    color: #ef4444;
    background: rgba(239, 68, 68, 0.1);
    padding: 1rem;
    border-radius: 12px;
    border: 1px solid rgba(239, 68, 68, 0.2);
  }

  /* Responsive Design */
  @media (max-width: 1024px) {
    .weather-layout {
      grid-template-columns: 1fr;
    }

    .forecast-grid {
      max-height: none;
    }
  }

  @media (max-width: 768px) {
    main {
      padding: 1rem;
    }

    #search-box {
      flex-direction: column;
      gap: 0.75rem;
      align-items: stretch;
    }

    .card {
      padding: 1rem;
    }

    .forecast-card {
      flex-direction: column;
      text-align: center;
      gap: 0.5rem;
    }

    .current-weather h2,
    .forecast-section h2 {
      font-size: 1.25rem;
    }

    #search-btn {
      padding: 0.75rem 1.25rem;
      font-size: 0.9rem;
    }
  }
</style>
