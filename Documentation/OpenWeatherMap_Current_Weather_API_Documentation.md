# OpenWeatherMap Current Weather Data API  
**Version:** 2.5  
**Base URL:** `https://api.openweathermap.org/data/2.5/weather`

## Introduction

The OpenWeatherMap Current Weather Data API provides real-time weather information for any location worldwide. This RESTful API is trusted by thousands of developers to deliver up-to-date weather conditions, temperature, humidity, wind, and more. With flexible query options, robust localization, and support for multiple data formats, the API is ideal for web, mobile, IoT, and enterprise applications.

## Table of Contents

1. [Getting Started](#getting-started)
2. [Authentication](#authentication)
3. [Endpoints](#endpoints)
    - [By City Name](#get-current-weather-by-city-name)
    - [By City ID](#get-current-weather-by-city-id)
    - [By Geographic Coordinates](#get-current-weather-by-geographic-coordinates)
    - [By ZIP Code](#get-current-weather-by-zip-code)
4. [Parameters](#parameters)
5. [Response Structure](#response-structure)
6. [Units & Localization](#units--localization)
7. [Weather Icons](#weather-icons)
8. [Error Handling](#error-handling)
9. [Rate Limits](#rate-limits)
10. [Best Practices](#best-practices)
11. [Code Examples](#code-examples)
12. [Additional Resources](#additional-resources)

## Getting Started

1. **Sign Up:**  
   Register for a free account at [OpenWeatherMap](https://home.openweathermap.org/users/sign_up).
2. **Get Your API Key:**  
   After email verification, navigate to your dashboard and copy your unique API key.
3. **Read Terms:**  
   Review the [Terms of Service](https://openweathermap.org/terms).

## Authentication

All requests require your API key, passed as the `appid` parameter in the query string.

**Example:**
```
GET /data/2.5/weather?q=London&appid=YOUR_API_KEY
```

- **Never expose your API key in public repositories or client-side code.**
- For production, consider using a backend proxy to secure your key.

## Endpoints

### Get Current Weather by City Name

**GET** `/data/2.5/weather`

| Parameter | Type   | Description                                  |
|-----------|--------|----------------------------------------------|
| q         | string | City name (optionally with country code, e.g. `London,uk`) |
| appid     | string | Your API key                                 |

**Example:**
```
GET /data/2.5/weather?q=Paris,fr&appid=YOUR_API_KEY
```

### Get Current Weather by City ID

Each city has a unique ID. Download the [city list](http://bulk.openweathermap.org/sample/) for reference.

| Parameter | Type   | Description      |
|-----------|--------|------------------|
| id        | int    | City ID          |
| appid     | string | Your API key     |

**Example:**
```
GET /data/2.5/weather?id=5128581&appid=YOUR_API_KEY
```

### Get Current Weather by Geographic Coordinates

| Parameter | Type   | Description      |
|-----------|--------|------------------|
| lat       | float  | Latitude         |
| lon       | float  | Longitude        |
| appid     | string | Your API key     |

**Example:**
```
GET /data/2.5/weather?lat=35.6895&lon=139.6917&appid=YOUR_API_KEY
```

### Get Current Weather by ZIP Code

| Parameter | Type   | Description      |
|-----------|--------|------------------|
| zip       | string | ZIP code, optionally with country code (e.g., `94040,us`) |
| appid     | string | Your API key     |

**Example:**
```
GET /data/2.5/weather?zip=94040,us&appid=YOUR_API_KEY
```

## Parameters

| Name   | Type   | Description                                         | Example         |
|--------|--------|-----------------------------------------------------|-----------------|
| q      | string | City name, optionally with country code             | `London,uk`     |
| id     | int    | City ID                                             | `2172797`       |
| lat    | float  | Latitude                                            | `35.6895`       |
| lon    | float  | Longitude                                           | `139.6917`      |
| zip    | string | ZIP/postal code, optionally with country code       | `94040,us`      |
| units  | string | `standard` (Kelvin, default), `metric` (Celsius), `imperial` (Fahrenheit) | `metric`        |
| lang   | string | Response language (ISO 639-1 code)                  | `es`            |
| mode   | string | Response format: `json` (default), `xml`, `html`    | `xml`           |
| appid  | string | Your API key                                        | `YOUR_API_KEY`  |

## Response Structure

**Example JSON Response:**
```json
{
  "coord": { "lon": 2.35, "lat": 48.85 },
  "weather": [
    {
      "id": 800,
      "main": "Clear",
      "description": "clear sky",
      "icon": "01d"
    }
  ],
  "base": "stations",
  "main": {
    "temp": 18.5,
    "feels_like": 18.3,
    "temp_min": 17.0,
    "temp_max": 20.0,
    "pressure": 1012,
    "humidity": 60
  },
  "visibility": 10000,
  "wind": { "speed": 4.1, "deg": 80 },
  "clouds": { "all": 0 },
  "dt": 1622210400,
  "sys": {
    "country": "FR",
    "sunrise": 1622168400,
    "sunset": 1622224800
  },
  "timezone": 7200,
  "id": 2988507,
  "name": "Paris",
  "cod": 200
}
```

### Key Fields

- `coord`: Coordinates of the city
- `weather`: Array of weather conditions (main, description, icon)
- `main`: Temperature, pressure, humidity, min/max temps
- `wind`: Wind speed and direction
- `clouds`: Cloudiness in %
- `sys`: Country, sunrise/sunset times (Unix UTC)
- `name`: City name
- `cod`: Status code

## Units & Localization

- **units**
  - `standard`: Kelvin (default)
  - `metric`: Celsius, wind speed in m/s
  - `imperial`: Fahrenheit, wind speed in mph

- **lang**
  - Use ISO 639-1 codes for localization (e.g., `en`, `es`, `fr`, `de`)
  - Weather descriptions will be translated accordingly

**Example:**
```
GET /data/2.5/weather?q=Madrid&units=metric&lang=es&appid=YOUR_API_KEY
```

## Weather Icons

The `icon` field in the response provides a code for weather icons.  
To display an icon, use:

```
https://openweathermap.org/img/wn/{icon}@2x.png
```
**Example:**  
`https://openweathermap.org/img/wn/01d@2x.png`

Refer to the [Weather Icon List](https://openweathermap.org/weather-conditions) for all codes.

## Error Handling

| Code | Meaning                     | Example Message           |
|------|-----------------------------|---------------------------|
| 200  | OK                          |                           |
| 401  | Unauthorized (invalid API key) | {"cod":401, "message":"Invalid API key"} |
| 404  | Not Found (city not found)  | {"cod":"404","message":"city not found"} |
| 429  | Too Many Requests (rate limit) | {"cod":429, "message":"Your account is temporary blocked"} |
| 500  | Internal Server Error       | {"cod":500, "message":"Internal server error"} |

**Error Example:**
```json
{
  "cod": "404",
  "message": "city not found"
}
```

Always check the `cod` and `message` fields in the response for error handling.

## Rate Limits

- **Free tier:** 60 requests/minute
- **Paid plans:** Higher limits available (see [Pricing](https://openweathermap.org/price))
- Exceeding the limit returns a `429 Too Many Requests` error

**Tip:**  
Cache weather data and throttle requests to avoid hitting rate limits.

## Best Practices

- **API Key Security:** Never expose your API key in public repositories. Use environment variables or backend proxies.
- **Caching:** Cache responses for at least 10 minutes to reduce API calls and improve performance.
- **Localization:** Set the `lang` parameter for user-friendly, localized weather descriptions.
- **Units:** Always specify `units` to avoid confusion (especially for international users).
- **Attribution:** Credit OpenWeatherMap as your data source in your application.
- **Error Handling:** Always check for and gracefully handle error responses.
- **HTTPS:** Always use HTTPS for all API requests.

## Code Examples

### Python

```python
import requests

API_KEY = "YOUR_API_KEY"
city = "Berlin"
url = f"https://api.openweathermap.org/data/2.5/weather?q={city}&units=metric&appid={API_KEY}"

response = requests.get(url)
data = response.json()

if response.status_code == 200:
    print("City:", data["name"])
    print("Temperature:", data["main"]["temp"], "°C")
    print("Weather:", data["weather"][0]["description"])
else:
    print("Error:", data["message"])
```

### JavaScript (Fetch API)

```javascript
const API_KEY = "YOUR_API_KEY";
const city = "New York";
const url = `https://api.openweathermap.org/data/2.5/weather?q=${city}&units=imperial&appid=${API_KEY}`;

fetch(url)
  .then(response => response.json())
  .then(data => {
    if (data.cod === 200) {
      console.log("City:", data.name);
      console.log("Temperature:", data.main.temp, "°F");
      console.log("Weather:", data.weather[0].description);
    } else {
      console.error("Error:", data.message);
    }
  });
```

### cURL

```bash
curl "https://api.openweathermap.org/data/2.5/weather?q=Tokyo&appid=YOUR_API_KEY&units=metric"
```

## Additional Resources

- [OpenWeatherMap API Documentation](https://openweathermap.org/current)
- [City List (IDs)](http://bulk.openweathermap.org/sample/)
- [Weather Icon List](https://openweathermap.org/weather-conditions)
- [API Pricing](https://openweathermap.org/price)
- [Terms of Service](https://openweathermap.org/terms)