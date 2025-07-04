<script>
	// Global variables
	const API_KEY = process.env.API_KEY;
	let search_input = "";
	let forecast_days = 5;
	let search_type = "current";
	let data = null;

	// async function to fetch weather data
	async function fetchData(city, endpoint, options = {}) {
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
		if (search_type === "current") {
			data = await fetchData(search_input.trim(), search_type);
		} else {
			data = await fetchData(search_input.trim(), search_type, { days: forecast_days });
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
				{:else if search_type === "forecast" && data.forecast !== undefined}
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
	/* Weather App Styles */
	* {
		margin: 0;
		padding: 0;
		box-sizing: border-box;
	}

	body, html{
		font-family:
			"Inter",
			-apple-system,
			BlinkMacSystemFont,
			"Segoe UI",
			sans-serif;
		background: linear-gradient(135deg, #0c0c0c 0%, #1a1a1a 100%);
		color: #e0e0e0;
		min-height: 100vh;
	}

	main {
		min-height: 100vh;
		min-width: 100vw;
		padding: 2rem;
		display: flex;
		justify-content: center;
		align-items: flex-start;
	}

	#main {
		width: 100%;
		max-width: 800px;
		background: rgba(20, 20, 20, 0.9);
		backdrop-filter: blur(20px);
		border: 1px solid rgba(255, 255, 255, 0.1);
		border-radius: 20px;
		padding: 2.5rem;
		box-shadow: 0 20px 40px rgba(0, 0, 0, 0.3);
	}

	/* Search Type Box */
	#search-type-box {
		background: rgba(30, 30, 30, 0.8);
		border: 1px solid rgba(255, 255, 255, 0.05);
		border-radius: 16px;
		padding: 1.5rem;
		margin-bottom: 2rem;
		display: flex;
		align-items: center;
		gap: 1rem;
		flex-wrap: wrap;
	}

	#search-type-box h4 {
		color: #ffffff;
		font-size: 1.1rem;
		font-weight: 600;
		margin: 0;
	}

	#search-type-box h5 {
		color: #b0b0b0;
		font-size: 0.9rem;
		font-weight: 500;
		margin: 0;
	}

	#search-menu {
		background: rgba(40, 40, 40, 0.9);
		border: 1px solid rgba(255, 255, 255, 0.1);
		border-radius: 8px;
		color: #ffffff;
		padding: 0.5rem 1rem;
		font-size: 0.9rem;
		outline: none;
		cursor: pointer;
		transition: all 0.3s ease;
	}

	#search-menu:hover {
		border-color: rgba(255, 255, 255, 0.2);
		background: rgba(50, 50, 50, 0.9);
	}

	#search-menu:focus {
		border-color: #4f46e5;
		box-shadow: 0 0 0 3px rgba(79, 70, 229, 0.1);
	}

	#days-input {
		background: rgba(40, 40, 40, 0.9);
		border: 1px solid rgba(255, 255, 255, 0.1);
		border-radius: 8px;
		color: #ffffff;
		padding: 0.5rem;
		width: 80px;
		font-size: 0.9rem;
		outline: none;
		transition: all 0.3s ease;
	}

	#days-input:hover {
		border-color: rgba(255, 255, 255, 0.2);
	}

	#days-input:focus {
		border-color: #4f46e5;
		box-shadow: 0 0 0 3px rgba(79, 70, 229, 0.1);
	}

	/* Search Box */
	#search-box {
		display: flex;
		gap: 1rem;
		margin-bottom: 2rem;
	}

	#search-input {
		flex: 1;
		background: rgba(30, 30, 30, 0.8);
		border: 1px solid rgba(255, 255, 255, 0.1);
		border-radius: 12px;
		color: #ffffff;
		padding: 1rem 1.5rem;
		font-size: 1rem;
		outline: none;
		transition: all 0.3s ease;
	}

	#search-input::placeholder {
		color: #666666;
	}

	#search-input:hover {
		border-color: rgba(255, 255, 255, 0.2);
		background: rgba(35, 35, 35, 0.8);
	}

	#search-input:focus {
		border-color: #4f46e5;
		box-shadow: 0 0 0 3px rgba(79, 70, 229, 0.1);
		background: rgba(35, 35, 35, 0.9);
	}

	#search-btn {
		background: linear-gradient(135deg, #4f46e5 0%, #7c3aed 100%);
		border: none;
		border-radius: 12px;
		color: #ffffff;
		padding: 1rem 2rem;
		font-size: 1rem;
		font-weight: 600;
		cursor: pointer;
		transition: all 0.3s ease;
		box-shadow: 0 4px 15px rgba(79, 70, 229, 0.3);
	}

	#search-btn:hover {
		transform: translateY(-2px);
		box-shadow: 0 8px 25px rgba(79, 70, 229, 0.4);
		background: linear-gradient(135deg, #5b52e8 0%, #8b44f0 100%);
	}

	#search-btn:active {
		transform: translateY(0);
	}

	/* Result Box */
	#result-box {
		background: rgba(25, 25, 25, 0.6);
		border: 1px solid rgba(255, 255, 255, 0.05);
		border-radius: 16px;
		padding: 2rem;
		min-height: 200px;
	}

	#result-box h2 {
		color: #ffffff;
		font-size: 1.8rem;
		font-weight: 700;
		margin-bottom: 1.5rem;
		text-align: center;
	}

	#result-box p {
		color: #ff6b6b;
		font-size: 1.1rem;
		text-align: center;
		padding: 2rem;
		background: rgba(255, 107, 107, 0.1);
		border: 1px solid rgba(255, 107, 107, 0.2);
		border-radius: 12px;
	}

	#result-box img {
		display: block;
		margin: 1rem auto;
		width: 80px;
		height: 80px;
		filter: drop-shadow(0 4px 8px rgba(0, 0, 0, 0.3));
	}

	/* Item Styles */
	.item {
		display: flex;
		justify-content: space-between;
		align-items: center;
		padding: 0.8rem 0;
		border-bottom: 1px solid rgba(255, 255, 255, 0.05);
		transition: all 0.3s ease;
	}

	.item:last-child {
		border-bottom: none;
	}

	.item:hover {
		background: rgba(255, 255, 255, 0.02);
		padding-left: 0.5rem;
		padding-right: 0.5rem;
		border-radius: 8px;
	}

	.item strong {
		color: #b0b0b0;
		font-weight: 600;
		font-size: 0.9rem;
	}

	.item span {
		color: #ffffff;
		font-weight: 500;
		font-size: 0.95rem;
	}

	/* Forecast Grid */
	.forecast-grid {
		display: grid;
		grid-template-columns: repeat(auto-fit, minmax(280px, 1fr));
		gap: 1.5rem;
		margin-top: 1rem;
	}

	.card {
		background: rgba(35, 35, 35, 0.8);
		border: 1px solid rgba(255, 255, 255, 0.08);
		border-radius: 16px;
		padding: 1.5rem;
		transition: all 0.3s ease;
		position: relative;
		overflow: hidden;
	}

	.card::before {
		content: "";
		position: absolute;
		top: 0;
		left: 0;
		right: 0;
		height: 3px;
		background: linear-gradient(90deg, #4f46e5, #7c3aed, #06b6d4);
		border-radius: 16px 16px 0 0;
	}

	.card:hover {
		transform: translateY(-4px);
		box-shadow: 0 12px 25px rgba(0, 0, 0, 0.3);
		border-color: rgba(255, 255, 255, 0.15);
	}

	.card img {
		width: 60px;
		height: 60px;
		margin: 0 auto 1rem;
		filter: drop-shadow(0 2px 4px rgba(0, 0, 0, 0.3));
	}

	.card p {
		text-align: center;
		color: #ffffff;
		font-weight: 600;
		margin-bottom: 1rem;
		font-size: 1rem;
		background: none;
		border: none;
		padding: 0;
	}

	.card .item {
		padding: 0.6rem 0;
		border-bottom: 1px solid rgba(255, 255, 255, 0.08);
	}

	.card .item:hover {
		background: rgba(255, 255, 255, 0.03);
	}

	/* Responsive Design */
	@media (max-width: 768px) {
		main {
			padding: 1rem;
		}

		#main {
			padding: 1.5rem;
		}

		#search-type-box {
			flex-direction: column;
			align-items: flex-start;
			gap: 0.8rem;
		}

		#search-box {
			flex-direction: column;
		}

		#search-btn {
			width: 100%;
		}

		.forecast-grid {
			grid-template-columns: 1fr;
		}
	}

	@media (max-width: 480px) {
		#main {
			padding: 1rem;
		}

		#result-box {
			padding: 1rem;
		}

		.item {
			flex-direction: column;
			align-items: flex-start;
			gap: 0.3rem;
		}
	}
</style>
