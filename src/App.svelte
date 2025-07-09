<script>
    // Global variables
    const API_KEY = process.env.API_KEY;
    let search_input = "";
    let loading = false;
    let currentData = null;
    let forecastData = null;

    // Get user location from ip
    window.addEventListener("load", async () => {
        loading = true;
        const ipData = await (await fetch("https://ipwho.is")).json();
        [currentData, forecastData] = await fetchData(
            `${ipData.latitude}, ${ipData.longitude}`,
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

    async function handleClick() {
        loading = true;
        // Fetch data and store it in variables
        [currentData, forecastData] = await fetchData(search_input.trim());
        loading = false;
    }
</script>

<main>
    <div id="main">
        <div id="search-box">
            <input
                type="text"
                name="search"
                id="search-input"
                placeholder="Type Here"
                bind:value={search_input}
            />
            <button type="submit" on:click={handleClick} id="search-btn"
                >Search</button
            >
        </div>

        <div id="result-box">
            {#if loading}
                <div class="loader">Fetching data...</div>
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
                            <img
                                src={currentData.current.condition.icon}
                                alt="Icon"
                            />
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
                                        <img
                                            src={day.day.condition.icon}
                                            alt="Icon"
                                        />
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

    /* Search Section */
    #search-box {
        display: flex;
        gap: 1rem;
        margin-bottom: 2rem;
        justify-content: center;
    }

    #search-input {
        flex: 1;
        max-width: 400px;
        padding: 0.875rem 1.25rem;
        background: rgba(255, 255, 255, 0.05);
        border: 1px solid rgba(255, 255, 255, 0.1);
        border-radius: 12px;
        color: #e4e4e7;
        font-size: 1rem;
        transition: all 0.3s ease;
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

    /* Loading State */
    .loader {
        text-align: center;
        padding: 3rem;
        color: rgba(228, 228, 231, 0.7);
        font-size: 1.1rem;
    }

    /* Error Messages */
    #result-box p {
        /* text-align: center; */
        color: #ef4444;
        background: rgba(239, 68, 68, 0.1);
        padding: 1rem;
        border-radius: 12px;
        border: 1px solid rgba(239, 68, 68, 0.2);
        /* margin: 2rem 0; */
    }

    /* Weather Layout */
    .weather-layout {
        display: grid;
        grid-template-columns: 1fr 1fr;
        gap: 2rem;
        margin-top: 2rem;
        align-items: start;
    }

    /* Card Base Styles */
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

    /* Current Weather Section */
    .current-weather {
        display: flex;
        flex-direction: column;
        justify-content: space-between;
        height: 100%;
    }

    .current-weather h2 {
        font-size: 1.5rem;
        font-weight: 600;
        margin-bottom: 1.5rem;
        color: #f8fafc;
        text-align: center;
    }

    .current-weather img {
        width: 80px;
        height: 80px;
        margin: 1rem 0;
        filter: drop-shadow(0 4px 8px rgba(0, 0, 0, 0.3));
    }

    .current-weather p {
        display: flex;
        justify-content: space-between;
        margin: 0.5rem 0;
        font-size: 0.95rem;
    }

    .current-weather p:first-of-type {
        font-size: 1.1rem;
        color: #94a3b8;
        margin-bottom: 1rem;
    }

    .current-weather strong {
        color: #cbd5e1;
        font-weight: 500;
    }

    /* Forecast Section */
    .forecast-section {
        display: flex;
        flex-direction: column;
        height: 100%;
    }

    .forecast-section h2 {
        font-size: 1.5rem;
        font-weight: 600;
        margin-bottom: 1.5rem;
        color: #f8fafc;
        text-align: center;
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
        align-items: center;
        justify-content: space-between;
        gap: 1rem;
        padding: 1rem;
        text-align: left;
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
        filter: drop-shadow(0 2px 4px rgba(0, 0, 0, 0.3));
    }

    .forecast-card p {
        font-size: 0.85rem;
        margin: 0.25rem 0;
    }

    .forecast-card p:first-of-type {
        color: #cbd5e1;
        font-weight: 500;
    }

    .forecast-card strong {
        color: #94a3b8;
        font-weight: 500;
    }

    /* Responsive Design */
    @media (max-width: 1024px) {
        .weather-layout {
            grid-template-columns: 1fr;
            gap: 1.5rem;
        }

        .forecast-grid {
            grid-template-columns: 1fr;
        }
    }

    @media (max-width: 768px) {
        main {
            padding: 1rem;
        }

        #search-box {
            flex-direction: column;
            align-items: stretch;
        }

        #search-input {
            max-width: none;
        }

        .forecast-card {
            grid-template-columns: 1fr;
            text-align: center;
            gap: 0.5rem;
        }

        .forecast-card h4 {
            min-width: auto;
        }
    }

    @media (max-width: 480px) {
        .current-weather,
        .forecast-card {
            padding: 1rem;
        }

        .current-weather h2,
        .forecast-section h2 {
            font-size: 1.25rem;
        }
    }
    @media (max-width: 768px) {
        .weather-layout {
            grid-template-columns: 1fr;
            gap: 1rem;
        }

        .card {
            padding: 1rem;
        }

        .current-weather,
        .forecast-section {
            padding: 1rem 0;
            text-align: center;
        }

        #search-box {
            flex-direction: column;
            gap: 0.75rem;
            align-items: stretch;
        }

        #search-input {
            width: 100%;
            max-width: none;
        }

        .forecast-grid {
            grid-template-columns: 1fr;
            gap: 0.75rem;
        }

        .forecast-card {
            grid-template-columns: 1fr;
            text-align: center;
            gap: 0.5rem;
        }

        .forecast-card h4 {
            min-width: auto;
        }

        main {
            padding: 1rem;
        }
    }

    @media (max-width: 480px) {
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
