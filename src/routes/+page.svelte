<script>
    var timeOfDay = getDayTime();
    // Import svelte on mount
    import { onMount } from 'svelte';
    onMount(() => {
        // Get weather data
        var weatherData = getWeatherData();
        function getWeatherData () {
            var weatherData;
            var xhr = new XMLHttpRequest();
            xhr.open("GET", "https://api.openweathermap.org/data/2.5/forecast?lat=34.0515&lon=84.0713&appid=97d22eb77a35ad108e900a017d29a38b", false);
            xhr.onload = function (e) {
                if (xhr.readyState === 4) {
                    if (xhr.status === 200) {
                        weatherData = JSON.parse(xhr.responseText);
                    } else {
                        console.error(xhr.statusText);
                    }
                }
            };
            xhr.onerror = function (e) {
                console.error(xhr.statusText);
            };
            xhr.send(null);
            return weatherData;
        }
    });
    function getDayTime () {
        // Get morning, evening, afternoon, or night
        var timeOfDay;
        var date = new Date();
        var hours = date.getHours();
        if (hours >= 5 && hours < 12) {
            timeOfDay = "morning";
        } else if (hours >= 12 && hours < 17) {
            timeOfDay = "afternoon";
        } else if (hours >= 17 && hours < 21) {
            timeOfDay = "evening";
        } else {
            timeOfDay = "night";
        }
        return timeOfDay;
    }
    var time;
    // Refresh the time every 50ms
    setInterval(function() {
        var date = new Date();
        var hours = date.getHours();
        var minutes = date.getMinutes();
        var seconds = date.getSeconds();
        // Add leading zeros
        if (hours < 10) hours = "0" + hours;
        if (minutes < 10) minutes = "0" + minutes;
        // Compose the string for display
        time = hours + ":" + minutes;
    }, 50);
    // 15 minute scripts
    setInterval(function() {
        timeOfDay = getDayTime()
        weatherData = getWeatherData()
    }, 900000);
</script>
<div>
    <nav class="tabs min left-align">
        <a id="mainTab" data-ui="#mainPage" class="active">Good {timeOfDay}</a>
        <a id="weatherTab" data-ui="#weatherPage">Weather</a>
        <a id="mediaTab" data-ui="#mediaPage">Media</a>
        <div class="max"></div>
        <a>{time}</a>
    </nav>
    <div class="page padding active bottom" id="mainPage">
        <article class="border">
            <h5>Weather is warming up...</h5>
            <p>Please wait</p>
        </article>
    </div>
    <div class="page padding bottom" id="weatherPage">

        <article class="border">
            <h5>Weather is warming up...</h5>
            <p>Please wait</p>
        </article>    </div>
    <div class="page padding bottom" id="mediaPage">
        <h5>Tab 3</h5>
    </div>
</div>