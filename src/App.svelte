<script>
	// Global variables
	const API_KEY = process.env.API_KEY;
	let search_input = "";
	let forecast_days = 5;
	let search_type = "current";
	let data = null;

	// async function to fetch weather data
	async function fetchData(city, endpoint, options = null) {
		let url;

		if (endpoint === "forecast" && options.days !== undefined) {
			url = `http://api.weatherapi.com/v1/${endpoint}.json?key=${API_KEY}&q=${city}&days=${options.days}`;
		} else {
			url = `http://api.weatherapi.com/v1/${endpoint}.json?key=${API_KEY}&q=${city}`;
		}

		// try {
		// 	const res = await fetch(url);
		// 	const json_data = await res.json();
		// 	if (!res.ok) {
		// 		throw new Error(
		// 			json_data.error?.message || "Failed to fetch data",
		// 		);
		// 	}
		// 	return json_data;
		// } catch (err) {
		// 	console.error("Fetch error:", err.message);
		// 	data = { error: { message: err.message } };
		// }
		return await (await fetch(url)).json();
	}

	async function handleClick() {
		const endpoint = search_type;
		// console.log(endpoint);
		const query = search_input;
		// console.log(query);
		if (endpoint === "current") {
			data = await fetchData(query, endpoint);
		} else {
			data = await fetchData(query, endpoint, { days: forecast_days });
		}
		console.log(data);
	}
</script>

<main>
	<div id="main">
		<div id="search-type-box">
			<h4>Select what to do</h4>
			<select bind:value={search_type} id="search-menu">
				<option value="current">Current</option>
				<option value="forecast">Forecast</option>
			</select>
			{#if search_type === "forecast"}
				<h5>Forecast to</h5>
				<input
					bind:value={forecast_days}
					type="number"
					name="days"
					id="days-input"
					min="1"
					max="14"
				/>
				<h5>Days</h5>
			{/if}
		</div>

		<div id="search-box">
			<input
				type="text"
				name="search"
				id="search-input"
				placeholder="Type Here"
				bind:value={search_input}
			/>
			<button on:click={handleClick} id="search-btn">Search</button>
		</div>

		<div id="result-box">
			{#if data !== null}
				{#if data.error}
					{#if data.error.code === 1006 || data.error.code === 1005}
						<p>No matching location found</p>
					{:else if data.error.code === 1003}
						<p>Please provide a city name or location</p>
					{/if}
				{:else if search_type === "current"}
					<h2>Current Weather</h2>
					<div class="item">
						<strong>Condition:</strong>
						<span>{data.current.condition.text}</span>
					</div>
					<img src={data.current.condition.icon} alt="icon" />
					<div class="item">
						<strong>Country:</strong>
						<span id="country">{data.location.country}</span>
					</div>
					<div class="item">
						<strong>Region:</strong>
						<span id="region">{data.location.region}</span>
					</div>
					<div class="item">
						<strong>City:</strong>
						<span id="name">{data.location.name}</span>
					</div>
					<div class="item">
						<strong>Time Zone:</strong>
						<span id="timezone">{data.location.tz_id}</span>
					</div>
					<div class="item">
						<strong>Local Time:</strong>
						<span id="localtime">{data.location.localtime}</span>
					</div>
					<div class="item">
						<strong>Temperature:</strong>
						<span id="temp">{data.current.temp_c} °C</span>
					</div>
					<div class="item">
						<strong>Feels Like:</strong>
						<span id="feelslike">{data.current.feelslike_c} °C</span
						>
					</div>
					<div class="item">
						<strong>Humidity:</strong>
						<span id="humidity">{data.current.humidity} %</span>
					</div>
					<div class="item">
						<strong>Wind Speed:</strong>
						<span id="wind">{data.current.wind_kph} kph</span>
					</div>
					<div class="item">
						<strong>UV Index:</strong>
						<span id="uv">{data.current.uv}</span>
					</div>
				{:else if search_type === "forecast"}
					<div class="forecast-grid">
						{#each data.forecast.forecastday as day}
							<div class="card">
								<img src={day.day.condition.icon} alt="icon" />
								<p>{day.day.condition.text}</p>
								<div class="item">
									<strong>Date:</strong>
									<span id="date">{day.date}</span>
								</div>
								<div class="item">
									<strong>Max Temp:</strong>
									<span id="date">{day.day.maxtemp_c} °C</span
									>
								</div>
								<div class="item">
									<strong>Min Temp:</strong>
									<span id="date">{day.day.mintemp_c} °C</span
									>
								</div>
								<div class="item">
									<strong>Avg Temp:</strong>
									<span id="date">{day.day.avgtemp_c} °C</span
									>
								</div>
								<div class="item">
									<strong>Max Wind:</strong>
									<span id="date"
										>{day.day.maxwind_kph} kph</span
									>
								</div>
								<div class="item">
									<strong>Avg Humidity:</strong>
									<span id="date"
										>{day.day.avghumidity} %</span
									>
								</div>
							</div>
						{/each}
					</div>
				{/if}
			{/if}
		</div>
	</div>
</main>

<style>

	body, html{
		background-color: #000000;
	}

	/* Main container */
	main {
		padding: 2rem;
		max-width: 800px;
		margin: 0 auto;
		font-family: "Segoe UI", sans-serif;
		color: #dbdbdb;
		background-color: #292929;
	}

	/* Layout */
	#main {
		display: flex;
		flex-direction: column;
		gap: 1.5rem;
	}

	/* Search and selection UI */
	#search-type-box,
	#search-box {
		display: flex;
		flex-direction: column;
		align-items: center;
		gap: 0.5rem;
	}

	/* Inputs and select dropdown */
	input,
	select {
		padding: 0.5rem;
		width: 100%;
		max-width: 300px;
		border-radius: 6px;
		border: 1px solid #ccc;
		font-size: 1rem;
		box-shadow: 0 1px 3px rgba(0, 0, 0, 0.1);
	}

	/* Button style */
	button {
		padding: 0.5rem 1.2rem;
		background-color: #007bff;
		color: #fff;
		font-size: 1rem;
		border: none;
		border-radius: 6px;
		cursor: pointer;
		transition: background-color 0.3s ease;
		box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
	}

	button:hover {
		background-color: #0056b3;
	}

	/* Result section */
	#result-box {
		display: flex;
		flex-direction: column;
		align-items: center;
		margin-top: 1rem;
	}

	/* Item row (label + value) */
	.item {
		display: flex;
		justify-content: space-between;
		width: 100%;
		max-width: 500px;
		margin: 0.3rem 0;
		padding: 0.2rem 0.5rem;
		background-color: #fff;
		border-radius: 5px;
		box-shadow: 0 1px 3px rgba(0, 0, 0, 0.05);
	}

	/* Forecast cards */
	.forecast-grid {
		display: grid;
		grid-template-columns: repeat(auto-fit, minmax(220px, 1fr));
		gap: 1rem;
		width: 100%;
	}

	.card {
		background: #ffffff;
		padding: 1rem;
		border-radius: 10px;
		box-shadow: 0 2px 6px rgba(0, 0, 0, 0.08);
		text-align: center;
		transition: transform 0.2s;
	}

	.card:hover {
		transform: scale(1.02);
	}

	/* Weather icons */
	.card img,
	#result-box img {
		width: 64px;
		height: 64px;
		margin: 0.5rem auto;
	}

	/* Loader spinner */
	/* .loader {
		border: 4px solid #f3f3f3;
		border-top: 4px solid #007bff;
		border-radius: 50%;
		width: 36px;
		height: 36px;
		animation: spin 1s linear infinite;
		margin: 1rem auto;
	} */

	/* @keyframes spin {
		0% {
			transform: rotate(0deg);
		}
		100% {
			transform: rotate(360deg);
		}
	} */

	/* Error and info messages */
	/* p {
		margin: 0.5rem 0;
		font-size: 1rem;
		color: #000000;
	} */

	/* p.error {
		color: red;
		font-weight: bold;
	} */
</style>
