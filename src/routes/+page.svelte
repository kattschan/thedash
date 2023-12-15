<script>
	import { xml2json } from 'xml-js';
	import { onMount } from 'svelte';

	let weatherData;
	let timeOfDay;
	let time;
	let weatherImage;
	let loading = true;
	let lockscreen = true;
	async function getWeatherDataToday() {
		const res = await fetch(
			`https://weather-broker-cdn.api.bbci.co.uk/en/forecast/rss/3day/4192289`
		);
		const xml = await res.text();
		const json = xml2json(xml, { compact: true });
		const parsed = JSON.parse(json);
		const data = parsed.rss.channel.item[0].title._text;
		return data.replace(/"/g, '');
	}
	async function getWeatherImage() {
		const res = await fetch(
			`https://weather-broker-cdn.api.bbci.co.uk/en/forecast/rss/3day/4192289`
		);
		const xml = await res.text();
		const json = xml2json(xml, { compact: true });
		const parsed = JSON.parse(json);
		return parsed.rss.channel.image.url._text;
	}
	function getDayTime() {
		// Get morning, evening, afternoon, or night
		var timeOfDay;
		var date = new Date();
		var hours = date.getHours();
		if (hours >= 5 && hours < 12) {
			timeOfDay = 'morning';
		} else if (hours >= 12 && hours < 17) {
			timeOfDay = 'afternoon';
		} else if (hours >= 17 && hours < 21) {
			timeOfDay = 'evening';
		} else {
			timeOfDay = 'night';
		}
		return timeOfDay;
	}
	function getBackgroundImage() {
		const images = {
			"image": "https://images.unsplash.com/photo-1695753723797-d505375d13a3?w=500&auto=format&fit=crop&q=60&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHx0b3BpYy1mZWVkfDZ8Ym84alFLVGFFMFl8fGVufDB8fHx8fA%3D%3D",
			"attribution": "Photo by <a href=\"https://unsplash.com/@ilona_a?utm_content=creditCopyText&utm_medium=referral&utm_source=unsplash\">Lena Polishko</a> on <a href=\"https://unsplash.com/photos/a-blue-staircase-leading-to-a-window-in-a-building-6MgoE_PhEMs?utm_content=creditCopyText&utm_medium=referral&utm_source=unsplash\">Unsplash</a>\n"
		}
		// Pick a random one
		const random = Math.floor(Math.random() * images.length);
		return images[random];
	}
	function toggleLockscreen() {
		lockscreen = lockscreen !== true;
	}
	// Refresh the time every 50ms
	setInterval(function () {
		var date = new Date();
		var hours = date.getHours();
		var minutes = date.getMinutes();
		var seconds = date.getSeconds();
		// Add leading zeros
		if (hours < 10) hours = '0' + hours;
		if (minutes < 10) minutes = '0' + minutes;
		// Compose the string for display
		time = hours + ':' + minutes;
	}, 50);
	// 15 minute scripts
	setInterval(async function () {
		timeOfDay = await getDayTime();
		weatherData = await getWeatherDataToday();
		weatherImage = await getWeatherImage();
	}, 900000);
	// onMount
	onMount(async function () {
		timeOfDay = await getDayTime();
		weatherData = await getWeatherDataToday();
		weatherImage = await getWeatherImage();

		var date = new Date();
		var hours = date.getHours();
		var minutes = date.getMinutes();
		var seconds = date.getSeconds();
		// Add leading zeros
		if (hours < 10) hours = '0' + hours;
		if (minutes < 10) minutes = '0' + minutes;
		// Compose the string for display
		time = hours + ':' + minutes;
		loading = false;
		// Every 50 ms
		setInterval(function () {
			// if lockscreen is true and anywhere is clicked
			if (lockscreen === true) {
				document.addEventListener('click', toggleLockscreen);
			} else {
				document.removeEventListener('click', toggleLockscreen);
			}
		}, 50);
	});
</script>

{#if loading === true}
	<h1>Loading...</h1>
{:else if lockscreen === true}
	<div class="lockscreen">
		<img style="width: 100vw; height: 100vh;" src="background.jpg" />
		<div class="absolute bottom left right padding bottom-shadow white-text">
			<h1>Good {timeOfDay}</h1>
			<h2>{time}</h2>
		</div>
		<div class="absolute bottom right padding bottom-shadow white-text">
			<h6>Touch anywhere to use System.</h6>
	</div>
	</div>
{:else}
	<div>
		<nav class="tabs min left-align">
			<a id="mainTab" data-ui="#mainPage" class="active">Good {timeOfDay}</a>
			<a id="weatherTab" data-ui="#weatherPage">Weather</a>
			<a id="mediaTab" data-ui="#mediaPage">Media</a>
			<div class="max"></div>
			<a>{time}</a>
		</nav>
		<div class="page padding active bottom" id="mainPage">
			{#await weatherData}
				<article class="border">
					<h5>Weather is starting up...</h5>
					<p>We're getting Weather ready for you</p>
				</article>
			{:then weatherData}
				<article class="border">
					<div class="row">
						<!-- Get everything before the colon and put it into the beginning p element -->
						<img class="circle large" src={JSON.stringify(weatherImage).replace(/"/g, '')} />
						<div class="max">
							<p>{JSON.stringify(weatherData.split(':')[0].toUpperCase()).replace(/"/g, '')}</p>
							<h5>{JSON.stringify(weatherData.split(':')[1].split(',')[0]).replace(/"/g, '')}</h5>
							<h6>{JSON.stringify(weatherData.split(',')[1]).replace(/"/g, '')}</h6>
						</div>
					</div>
				</article>
			{:catch error}
				<p>{error.message}</p>
			{/await}
		</div>
		<div class="page padding bottom" id="weatherPage">
			<article class="border">
				<h5>Weather is warming up...</h5>
				<p>Please wait</p>
			</article>
		</div>
		<div class="page padding bottom" id="mediaPage">
			<article class="border">
				<h5>You think I'm adding Spotify to this??</h5>
				<p>LMAOOOOO</p>
			</article>

		</div>
	</div>
{/if}
