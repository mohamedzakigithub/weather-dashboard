# Weather dashboard

## Description

This app access the open weather map API to retrieve current weather and forecast. The app uses local storage to store recently searched cities.

[Click here to the view the deployed website on github-pages](https://mohamedzakigithub.github.io/weather-dashboard/)

## Table of Contents

- [App demo](#App-demo)
- [Files](#Files)
- [HTML](#HTML)
- [JavaScript](#JavaScript)

## App demo

![](img/app.gif)

## Files

The app consists of three files (index.html, style.css, script.js), The html file holds the page layout and main components of text input, buttons and cards. The app uses Bootstrap for styling . The script.js file contains the app functionality to query the API and handle local storage.

## HTML

The html file contains the search input and button elements and the layout containing the elements that will display the received data from the API

## JavaScript

The script.js file holds the app logic and the functions to call and process the API ajax calls.

```javascript
// API endpoint URLs declaration
let weatherQueryURL = "https://api.openweathermap.org/data/2.5/weather?q=";
let forecastQueryURL = "https://api.openweathermap.org/data/2.5/forecast?q=";
let uviQueryURL = "https://api.openweathermap.org/data/2.5/uvi?";
// API hey declaration
let APIkey = "80fd36611a557ef9b88673bb16c8af2c";
// load local storage object for recent searched cities
let recentCities = JSON.parse(localStorage.getItem("recentCities")) || [];

// Add event listener for search button
$("#searchBtn").on("click", getData);

// Call the updateRecent function on page load
updateRecent();

// The updateRecent function adds buttons for recent cities, assign data-city variable for each button and
// add event listener to the buttons after creation.
function updateRecent() {
  $("#recent").empty();
  recentCities.forEach(function(item) {
    listItem = $("<li>").addClass("list-group-item");
    button = $("<button>").addClass("btn btn-block btn-primary recentBtn");
    button.text(item.city + ", " + item.country);
    button.attr("data-city", item.city + ", " + item.country);
    listItem.append(button);
    $("#recent").append(listItem);
  });
  $(".recentBtn").on("click", function() {
    $("#cityInput").val($(this).attr("data-city"));
    getData();
  });
}

// The addHistory function adds recent city to the local storage after taking into account maximum history array of 5 entries
function addHistory(recentCity) {
  if (recentCities.length < 5) {
    recentCities.unshift(recentCity);
  } else {
    recentCities.unshift(recentCity);
    recentCities.pop();
  }
  localStorage.setItem("recentCities", JSON.stringify(recentCities));
  updateRecent();
}

// The getData function creates tow ajax calls to the weather API and the forecast API then updates the elements on the page
// with the retrieved data.
function getData() {
  let city = $("#cityInput").val();
  $.ajax({
    url: weatherQueryURL + city + "&appid=" + APIkey + "&units=metric",
    method: "GET"
  })
    .then(function(response) {
      let recentCity = {};
      recentCity.city = response.name;
      recentCity.country = response.sys.country;
      addHistory(recentCity);
      let coords = response.coord;
      let icon = response.weather[0].icon;
      let currentDateTime = moment()
        .utc()
        .add(response.timezone, "s")
        .format("ddd, MMM Do HH:mm");
      $("#city").text(response.name + ", " + response.sys.country);
      $("#icon").attr(
        "src",
        "http://openweathermap.org/img/wn/" + icon + ".png"
      );
      $("#local").text(" " + currentDateTime);
      $("#temperature").text(
        "Temperature: " + response.main.temp.toFixed(0) + " ℃"
      );
      $("#humidity").text("Humidity: " + response.main.humidity + " %");
      $("#wind").text("Wind Speed: " + response.wind.speed + " m/sec");
      getUVI(coords);
    })
    .catch(function(error) {
      alert(error.responseJSON.message);
      location.reload();
    });
  $.ajax({
    url: forecastQueryURL + city + "&appid=" + APIkey + "&units=metric",
    method: "GET"
  }).then(function(response) {
    let weatherArr = [];
    for (let i = 0; i < response.list.length; i++) {
      let weatherObj = {};
      weatherObj.date = moment(
        (response.list[i].dt + response.city.timezone) * 1000
      )
        .utc()
        .format("DD/MM/YYYY");
      weatherObj.temp = response.list[i].main.temp;
      weatherObj.humidity = response.list[i].main.humidity;
      weatherObj.icon = response.list[i].weather[0].icon;
      weatherArr.push(weatherObj);
    }
    for (let i = 1; i <= 5; i++) {
      let card = $(".card[data-id=" + i + "] .card-body");
      let tempsDay = [];
      let humDay = [];
      let iconDay = [];
      let cardDate = moment()
        .utc()
        .add(response.city.timezone, "s")
        .add(i, "d")
        .format("DD/MM/YYYY");
      weatherArr.forEach(function(item) {
        if (item.date === cardDate) {
          tempsDay.push(item.temp);
          humDay.push(item.humidity);
          iconDay.push(item.icon);
        }
      });
      let maxTemp = Math.round(Math.max(...tempsDay));
      let minTemp = Math.round(Math.min(...tempsDay));
      let avgHum = humDay.reduce((a, b) => a + b, 0) / humDay.length;
      card.find(".date").text(cardDate);
      card.find(".temperature").text("Temp: " + minTemp + " / " + maxTemp);
      card.find(".humidity").text("Humidity: " + Math.round(avgHum));
      card
        .find("img")
        .attr(
          "src",
          "http://openweathermap.org/img/wn/" +
            iconDay[Math.floor(iconDay.length / 2)] +
            ".png"
        );
    }
  });
}

// the getUVI function creates an ajax call to the UV index endpoint and retrieves the UV index, update the element and sets the
// background color according to the value to reflect the severity level.
function getUVI(coords) {
  let lat = coords.lat;
  let lon = coords.lon;
  $.ajax({
    url: uviQueryURL + "appid=" + APIkey + "&lat=" + lat + "&lon=" + lon,
    method: "GET"
  }).then(function(response) {
    let uvi = response.value;
    let color = "";
    switch (true) {
      case uvi <= 3:
        color = "green";
        break;
      case uvi > 3 && uvi <= 6:
        color = "yellow";
        break;
      case uvi > 6 && uvi <= 8:
        color = " orange";
        break;
      case uvi > 8 && uvi <= 11:
        color = "red";
        break;
      case uvi > 11:
        color = "violet";
        break;
    }
    $("#uvi").text(" " + uvi);
    $("#uvi").css("background-color", color);
  });
}
```
