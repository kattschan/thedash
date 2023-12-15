<script>
	import {xml2json} from 'xml-js'
	import {onMount} from 'svelte';

	let weatherData;
	let timeOfDay;
	let loading = true;
	async function getWeatherDataToday() {
		const res = await fetch(`https://weather-broker-cdn.api.bbci.co.uk/en/forecast/rss/3day/4192289`);
		const xml = await res.text();
		const json = xml2json(xml, {compact: true})
		const parsed = JSON.parse(json);
		const data = parsed.rss.channel.item[0].title._text;
		return data;
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
	var time;
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
	}, 900000);
	// onMount
	onMount(async function () {
		timeOfDay = await getDayTime();
		weatherData = await getWeatherDataToday();
		loading = false;
	});
</script>
{#if loading === true}
	<h1>Loading...</h1>
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
				<h5>{JSON.stringify(weatherData)}</h5>
				<p>Weather data loaded</p>
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
		<h5>Tab 3</h5>
	</div>
</div>
{/if}