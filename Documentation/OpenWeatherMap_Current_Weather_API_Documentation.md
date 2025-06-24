# OpenWeatherMap Current Weather Data API - Comprehensive Documentation

**Version:** 2.5  
**Base URL:** `https://api.openweathermap.org/data/2.5/weather`

## Introduction

The OpenWeatherMap Current Weather Data API stands as one of the most reliable and comprehensive weather data services available to developers worldwide. This RESTful API delivers real-time meteorological information for virtually any location on Earth, serving millions of requests daily across web applications, mobile apps, IoT devices, and enterprise systems. 

The API provides access to current weather conditions including temperature, atmospheric pressure, humidity, wind patterns, cloud coverage, precipitation data, and visibility measurements. With its robust infrastructure and global data collection network, OpenWeatherMap ensures high availability and accuracy for mission-critical applications ranging from agricultural monitoring to smart city management.

## Table of Contents

1. [Getting Started](#getting-started)
2. [Authentication & Security](#authentication--security)
3. [API Endpoints](#api-endpoints)
4. [Request Parameters](#request-parameters)
5. [Response Structure & Data Types](#response-structure--data-types)
6. [Units & Localization](#units--localization)
7. [Weather Icons & Visual Elements](#weather-icons--visual-elements)
8. [Error Handling & Status Codes](#error-handling--status-codes)
9. [Rate Limits & Performance](#rate-limits--performance)
10. [Best Practices & Optimization](#best-practices--optimization)
11. [Code Examples & SDK Integration](#code-examples--sdk-integration)
12. [Advanced Features](#advanced-features)
13. [Troubleshooting](#troubleshooting)
14. [Additional Resources](#additional-resources)

## Getting Started

### Account Registration and Setup

1. **Create Your Account:**  
   Navigate to the [OpenWeatherMap registration page](https://home.openweathermap.org/users/sign_up) and create a free developer account. The registration process requires email verification and acceptance of the terms of service.

2. **API Key Generation:**  
   Once your email is verified, access your developer dashboard where you'll find your unique API key. This key serves as your authentication credential for all API requests and should be treated as sensitive information.

3. **Plan Selection:**  
   Review available subscription plans to determine the best fit for your usage requirements. The free tier provides 60 requests per minute, while paid plans offer higher rate limits and additional features.

4. **Terms and Compliance:**  
   Carefully review the [Terms of Service](https://openweathermap.org/terms) to ensure compliance with usage policies, attribution requirements, and data licensing terms.

### First API Call

Before diving into complex implementations, test your setup with a simple API call:

```bash
curl "https://api.openweathermap.org/data/2.5/weather?q=London&appid=YOUR_API_KEY"
```

A successful response indicates your API key is active and properly configured.

## Authentication & Security

### API Key Management

The OpenWeatherMap API uses API key-based authentication passed via the `appid` parameter in your request URL. This method provides a balance between security and ease of implementation while allowing for request tracking and rate limiting.

**Security Best Practices:**

- **Environment Variables:** Store your API key in environment variables rather than hardcoding it in your application source code
- **Backend Proxy:** For client-side applications, implement a backend proxy to hide your API key from public exposure
- **Key Rotation:** Regularly rotate your API keys, especially in production environments
- **Access Monitoring:** Monitor your API usage through the dashboard to detect unauthorized access

**Example Secure Implementation:**

```python
import os
import requests

API_KEY = os.environ.get('OPENWEATHER_API_KEY')
if not API_KEY:
    raise ValueError("API key not found in environment variables")

def get_weather(city):
    url = f"https://api.openweathermap.org/data/2.5/weather"
    params = {
        'q': city,
        'appid': API_KEY,
        'units': 'metric'
    }
    return requests.get(url, params=params)
```

### HTTPS and Data Transmission

All API requests must use HTTPS to ensure encrypted data transmission. This protects both your API key and the weather data during transit, preventing man-in-the-middle attacks and data interception.

## API Endpoints

The OpenWeatherMap API offers multiple query methods to accommodate different use cases and data availability scenarios.

### Get Current Weather by City Name

**Endpoint:** `GET /data/2.5/weather`

This is the most commonly used endpoint, allowing queries by city name with optional country specification.

| Parameter | Type   | Required | Description |
|-----------|--------|----------|-------------|
| q         | string | Yes      | City name, optionally with state and country codes |
| appid     | string | Yes      | Your API key |

**Query Format Options:**
- `London` - City name only
- `London,UK` - City with country code
- `London,GB` - Alternative country code format
- `New York,NY,US` - City with state and country (US cities)

**Example Requests:**
```bash
# Basic city query
GET /data/2.5/weather?q=Paris&appid=YOUR_API_KEY

# City with country
GET /data/2.5/weather?q=Tokyo,JP&appid=YOUR_API_KEY

# US city with state
GET /data/2.5/weather?q=Austin,TX,US&appid=YOUR_API_KEY
```

### Get Current Weather by City ID

**Endpoint:** `GET /data/2.5/weather`

City IDs provide the most precise location identification, eliminating ambiguity when multiple cities share the same name.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| id        | int  | Yes      | Unique city identifier |
| appid     | string | Yes    | Your API key |

**Advantages:**
- Eliminates name ambiguity
- Faster processing
- More reliable for automated systems
- Consistent across different languages

**Example:**
```bash
GET /data/2.5/weather?id=2988507&appid=YOUR_API_KEY  # Paris, France
```

**Finding City IDs:**
Download the comprehensive [city list](http://bulk.openweathermap.org/sample/) containing over 200,000 cities with their corresponding IDs.

### Get Current Weather by Geographic Coordinates

**Endpoint:** `GET /data/2.5/weather`

This endpoint is ideal for GPS-enabled applications, IoT devices, and location-based services.

| Parameter | Type  | Required | Description |
|-----------|-------|----------|-------------|
| lat       | float | Yes      | Latitude (-90 to 90) |
| lon       | float | Yes      | Longitude (-180 to 180) |
| appid     | string | Yes     | Your API key |

**Coordinate Precision:**
- Use at least 4 decimal places for city-level accuracy
- 6+ decimal places for neighborhood-level precision
- Higher precision may not yield different results due to data grid resolution

**Example:**
```bash
# Tokyo coordinates
GET /data/2.5/weather?lat=35.6762&lon=139.6503&appid=YOUR_API_KEY
```

### Get Current Weather by ZIP/Postal Code

**Endpoint:** `GET /data/2.5/weather`

Convenient for applications that collect user addresses or integrate with postal services.

| Parameter | Type   | Required | Description |
|-----------|--------|----------|-------------|
| zip       | string | Yes      | ZIP/postal code with optional country |
| appid     | string | Yes      | Your API key |

**Supported Formats:**
- `94040` - ZIP code only (defaults to US)
- `94040,US` - ZIP with country code
- `SW1A 1AA,GB` - UK postal code
- `75001,FR` - French postal code

**Example:**
```bash
GET /data/2.5/weather?zip=10001,US&appid=YOUR_API_KEY  # New York, NY
```

## Request Parameters

### Core Parameters

| Parameter | Type   | Required | Default | Description |
|-----------|--------|----------|---------|-------------|
| appid     | string | Yes      | -       | Your API authentication key |
| q         | string | Conditional | -    | City name query string |
| id        | int    | Conditional | -    | City ID for precise identification |
| lat       | float  | Conditional | -    | Latitude coordinate |
| lon       | float  | Conditional | -    | Longitude coordinate |
| zip       | string | Conditional | -    | ZIP/postal code |

### Optional Parameters

| Parameter | Type   | Default | Options | Description |
|-----------|--------|---------|---------|-------------|
| units     | string | standard | standard, metric, imperial | Temperature and measurement units |
| lang      | string | en      | ISO 639-1 codes | Response language |
| mode      | string | json    | json, xml, html | Response format |

### Units System Details

**Standard (Default):**
- Temperature: Kelvin
- Wind Speed: meter/sec
- Pressure: hPa

**Metric:**
- Temperature: Celsius
- Wind Speed: meter/sec
- Pressure: hPa

**Imperial:**
- Temperature: Fahrenheit
- Wind Speed: miles/hour
- Pressure: hPa

### Language Support

The API supports over 40 languages for weather descriptions. Common language codes include:

| Code | Language | Code | Language |
|------|----------|------|----------|
| en   | English  | es   | Spanish  |
| fr   | French   | de   | German   |
| it   | Italian  | pt   | Portuguese |
| ru   | Russian  | ja   | Japanese |
| zh_cn| Chinese Simplified | zh_tw | Chinese Traditional |

## Response Structure & Data Types

### Complete Response Schema

The API returns a comprehensive JSON object containing multiple data categories:

```json
{
  "coord": {
    "lon": 2.3488,
    "lat": 48.8534
  },
  "weather": [
    {
      "id": 803,
      "main": "Clouds",
      "description": "broken clouds",
      "icon": "04d"
    }
  ],
  "base": "stations",
  "main": {
    "temp": 22.5,
    "feels_like": 22.8,
    "temp_min": 20.0,
    "temp_max": 25.0,
    "pressure": 1013,
    "humidity": 65,
    "sea_level": 1013,
    "grnd_level": 1011
  },
  "visibility": 10000,
  "wind": {
    "speed": 3.6,
    "deg": 230,
    "gust": 5.1
  },
  "clouds": {
    "all": 75
  },
  "rain": {
    "1h": 0.5,
    "3h": 1.2
  },
  "snow": {
    "1h": 0,
    "3h": 0
  },
  "dt": 1646143200,
  "sys": {
    "type": 2,
    "id": 2041230,
    "country": "FR",
    "sunrise": 1646117400,
    "sunset": 1646158800
  },
  "timezone": 3600,
  "id": 2988507,
  "name": "Paris",
  "cod": 200
}
```

### Field Descriptions

**Coordinate Information:**
- `coord.lon`: Longitude of the location
- `coord.lat`: Latitude of the location

**Weather Conditions:**
- `weather[].id`: Weather condition ID (see weather codes)
- `weather[].main`: Group of weather parameters
- `weather[].description`: Weather condition description
- `weather[].icon`: Weather icon ID

**Temperature and Atmospheric Data:**
- `main.temp`: Current temperature
- `main.feels_like`: Perceived temperature accounting for wind and humidity
- `main.temp_min`: Minimum temperature (for large areas)
- `main.temp_max`: Maximum temperature (for large areas)
- `main.pressure`: Atmospheric pressure at sea level (hPa)
- `main.humidity`: Humidity percentage
- `main.sea_level`: Atmospheric pressure at sea level (hPa)
- `main.grnd_level`: Atmospheric pressure at ground level (hPa)

**Visibility and Wind:**
- `visibility`: Visibility in meters (max 10km)
- `wind.speed`: Wind speed
- `wind.deg`: Wind direction in degrees
- `wind.gust`: Wind gust speed

**Precipitation and Clouds:**
- `clouds.all`: Cloudiness percentage
- `rain.1h`: Rain volume for last hour (mm)
- `rain.3h`: Rain volume for last 3 hours (mm)
- `snow.1h`: Snow volume for last hour (mm)
- `snow.3h`: Snow volume for last 3 hours (mm)

**System Information:**
- `dt`: Data calculation time (Unix UTC)
- `sys.country`: Country code
- `sys.sunrise`: Sunrise time (Unix UTC)
- `sys.sunset`: Sunset time (Unix UTC)
- `timezone`: Timezone offset from UTC (seconds)
- `name`: City name
- `cod`: Internal parameter/HTTP status code

## Units & Localization

### Temperature Conversions

Understanding temperature conversions is crucial for global applications:

**Kelvin to Celsius:** `°C = K - 273.15`
**Kelvin to Fahrenheit:** `°F = (K - 273.15) × 9/5 + 32`
**Celsius to Fahrenheit:** `°F = °C × 9/5 + 32`

### Wind Speed Conversions

**Meter/second to:**
- Miles/hour: `mph = m/s × 2.237`
- Kilometers/hour: `km/h = m/s × 3.6`
- Knots: `knots = m/s × 1.944`

### Pressure Conversions

**hPa (hectopascals) to:**
- Inches of Mercury: `inHg = hPa × 0.02953`
- Millimeters of Mercury: `mmHg = hPa × 0.7501`
- Millibars: `mb = hPa` (equivalent)

### Localization Best Practices

Implement proper localization by:

1. **Detecting User Locale:** Use browser settings or user preferences
2. **Setting Appropriate Units:** Match regional standards (Celsius for metric countries, Fahrenheit for US)
3. **Language Selection:** Provide weather descriptions in user's preferred language
4. **Date/Time Formatting:** Format timestamps according to local conventions

## Weather Icons & Visual Elements

### Icon System

The weather icon system uses a standardized code format: `{code}{period}.png`

**Period Indicators:**
- `d`: Day
- `n`: Night

**Common Icon Codes:**
- `01d/01n`: Clear sky
- `02d/02n`: Few clouds
- `03d/03n`: Scattered clouds
- `04d/04n`: Broken clouds
- `09d/09n`: Shower rain
- `10d/10n`: Rain
- `11d/11n`: Thunderstorm
- `13d/13n`: Snow
- `50d/50n`: Mist/Fog

### Icon URL Construction

**Standard Size (50x50px):**
```
https://openweathermap.org/img/w/{icon}.png
```

**High Resolution (100x100px):**
```
https://openweathermap.org/img/wn/{icon}@2x.png
```

**Quad Resolution (200x200px):**
```
https://openweathermap.org/img/wn/{icon}@4x.png
```

### Custom Icon Implementation

For custom weather displays, you can map weather condition IDs to your own icons:

```javascript
const weatherIconMap = {
  200: 'thunderstorm-rain',
  201: 'thunderstorm-rain',
  202: 'thunderstorm-rain-heavy',
  // ... more mappings
  800: 'clear-sky',
  801: 'few-clouds',
  802: 'scattered-clouds'
};

function getCustomIcon(weatherId) {
  return weatherIconMap[weatherId] || 'default-weather';
}
```

## Error Handling & Status Codes

### HTTP Status Codes

| Code | Status | Description | Action Required |
|------|--------|-------------|-----------------|
| 200  | OK | Request successful | Process data normally |
| 401  | Unauthorized | Invalid API key | Check API key validity |
| 404  | Not Found | Location not found | Verify location parameters |
| 429  | Too Many Requests | Rate limit exceeded | Implement backoff strategy |
| 500  | Internal Server Error | Server error | Retry after delay |
| 502  | Bad Gateway | Temporary server issue | Retry with exponential backoff |
| 503  | Service Unavailable | Service maintenance | Check service status |

### Error Response Format

Error responses follow a consistent structure:

```json
{
  "cod": "404",
  "message": "city not found"
}
```

### Comprehensive Error Handling Strategy

```python
import requests
import time
from requests.adapters import HTTPAdapter
from requests.packages.urllib3.util.retry import Retry

class WeatherAPIClient:
    def __init__(self, api_key):
        self.api_key = api_key
        self.base_url = "https://api.openweathermap.org/data/2.5/weather"
        self.session = self._create_session()
    
    def _create_session(self):
        session = requests.Session()
        retry_strategy = Retry(
            total=3,
            backoff_factor=1,
            status_forcelist=[429, 500, 502, 503, 504]
        )
        adapter = HTTPAdapter(max_retries=retry_strategy)
        session.mount("http://", adapter)
        session.mount("https://", adapter)
        return session
    
    def get_weather(self, **params):
        params['appid'] = self.api_key
        
        try:
            response = self.session.get(self.base_url, params=params, timeout=10)
            data = response.json()
            
            if response.status_code == 200:
                return {'success': True, 'data': data}
            else:
                return {
                    'success': False,
                    'error_code': data.get('cod'),
                    'error_message': data.get('message', 'Unknown error')
                }
                
        except requests.exceptions.Timeout:
            return {'success': False, 'error_message': 'Request timeout'}
        except requests.exceptions.ConnectionError:
            return {'success': False, 'error_message': 'Connection error'}
        except requests.exceptions.RequestException as e:
            return {'success': False, 'error_message': f'Request failed: {str(e)}'}
```

## Rate Limits & Performance

### Rate Limit Tiers

**Free Tier:**
- 60 calls per minute
- 1,000 calls per day
- Standard data access

**Startup Tier ($40/month):**
- 600 calls per minute
- 100,000 calls per month
- Priority support

**Developer Tier ($180/month):**
- 3,000 calls per minute
- 300,000 calls per month
- Advanced features

### Rate Limit Headers

Monitor your usage through response headers:

```
X-RateLimit-Limit: 60
X-RateLimit-Remaining: 59
X-RateLimit-Reset: 1646143260
```

### Performance Optimization Strategies

**1. Intelligent Caching:**

```python
import time
from functools import wraps

def cache_weather_data(expiry_minutes=10):
    cache = {}
    
    def decorator(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            # Create cache key from arguments
            cache_key = f"{args}_{sorted(kwargs.items())}"
            current_time = time.time()
            
            # Check if cached data exists and is still valid
            if (cache_key in cache and 
                current_time - cache[cache_key]['timestamp']  self.min_interval
    
    def poll_weather(self):
        for location in self.locations:
            if self.should_update(location):
                weather_data = self.api_client.get_weather(q=location)
                self.process_weather_data(location, weather_data)
                self.last_update[location] = time.time()
                time.sleep(1)  # Rate limiting courtesy
```

## Best Practices & Optimization

### Security Best Practices

**1. API Key Protection:**
```bash
# Environment variable setup
export OPENWEATHER_API_KEY="your_api_key_here"

# .env file (never commit to version control)
OPENWEATHER_API_KEY=your_api_key_here

# Docker secrets
docker secret create openweather_key /path/to/key/file
```

**2. Input Validation:**
```python
import re

def validate_coordinates(lat, lon):
    try:
        lat_float = float(lat)
        lon_float = float(lon)
        if not (-90  100:
        raise ValueError("Invalid city name")
    
    # Remove potentially harmful characters
    sanitized = re.sub(r'[<>\"\'&]', '', city)
    return sanitized.strip()
```

### Performance Monitoring

**1. Response Time Tracking:**
```python
import time
import statistics

class PerformanceMonitor:
    def __init__(self):
        self.response_times = []
        self.error_count = 0
        self.success_count = 0
    
    def record_request(self, response_time, success):
        self.response_times.append(response_time)
        if success:
            self.success_count += 1
        else:
            self.error_count += 1
    
    def get_stats(self):
        if not self.response_times:
            return None
        
        return {
            'avg_response_time': statistics.mean(self.response_times),
            'median_response_time': statistics.median(self.response_times),
            'success_rate': self.success_count / (self.success_count + self.error_count),
            'total_requests': len(self.response_times)
        }
```

### Data Quality and Validation

**1. Response Validation:**
```python
def validate_weather_response(data):
    required_fields = ['name', 'main', 'weather', 'coord']
    missing_fields = [field for field in required_fields if field not in data]
    
    if missing_fields:
        raise ValueError(f"Missing required fields: {missing_fields}")
    
    # Validate temperature ranges (basic sanity check)
    temp = data['main'].get('temp')
    if temp is not None:
        if temp  400:  # Kelvin range check
            raise ValueError(f"Temperature {temp}K outside valid range")
    
    return True
```

## Code Examples & SDK Integration

### Python Examples

**1. Asynchronous Requests:**
```python
import aiohttp
import asyncio

async def fetch_weather_async(session, city, api_key):
    url = "https://api.openweathermap.org/data/2.5/weather"
    params = {'q': city, 'appid': api_key, 'units': 'metric'}
    
    try:
        async with session.get(url, params=params) as response:
            if response.status == 200:
                return await response.json()
            else:
                return {'error': f'HTTP {response.status}'}
    except Exception as e:
        return {'error': str(e)}

async def get_multiple_cities_weather(cities, api_key):
    async with aiohttp.ClientSession() as session:
        tasks = [fetch_weather_async(session, city, api_key) for city in cities]
        results = await asyncio.gather(*tasks)
        return dict(zip(cities, results))

# Usage
cities = ['London', 'Paris', 'Tokyo', 'New York']
weather_data = asyncio.run(get_multiple_cities_weather(cities, API_KEY))
```

**2. Django Integration:**
```python
# models.py
from django.db import models
from django.utils import timezone

class WeatherData(models.Model):
    city_name = models.CharField(max_length=100)
    temperature = models.FloatField()
    humidity = models.IntegerField()
    description = models.CharField(max_length=200)
    updated_at = models.DateTimeField(default=timezone.now)
    
    class Meta:
        ordering = ['-updated_at']

# views.py
from django.http import JsonResponse
from django.views.decorators.cache import cache_page
import requests

@cache_page(60 * 15)  # Cache for 15 minutes
def weather_api_view(request, city):
    api_key = settings.OPENWEATHER_API_KEY
    url = f"https://api.openweathermap.org/data/2.5/weather"
    
    params = {
        'q': city,
        'appid': api_key,
        'units': 'metric'
    }
    
    try:
        response = requests.get(url, params=params, timeout=10)
        data = response.json()
        
        if response.status_code == 200:
            # Save to database
            weather_obj, created = WeatherData.objects.update_or_create(
                city_name=city,
                defaults={
                    'temperature': data['main']['temp'],
                    'humidity': data['main']['humidity'],
                    'description': data['weather'][0]['description']
                }
            )
            return JsonResponse(data)
        else:
            return JsonResponse({'error': data.get('message')}, status=400)
            
    except requests.RequestException as e:
        return JsonResponse({'error': 'API request failed'}, status=500)
```

### JavaScript/Node.js Examples

**1. Express.js API Wrapper:**
```javascript
const express = require('express');
const axios = require('axios');
const NodeCache = require('node-cache');

const app = express();
const cache = new NodeCache({ stdTTL: 600 }); // 10 minutes cache

class WeatherService {
    constructor(apiKey) {
        this.apiKey = apiKey;
        this.baseURL = 'https://api.openweathermap.org/data/2.5/weather';
    }
    
    async getWeather(query, options = {}) {
        const cacheKey = `weather_${JSON.stringify(query)}_${JSON.stringify(options)}`;
        
        // Check cache first
        const cachedData = cache.get(cacheKey);
        if (cachedData) {
            return { ...cachedData, cached: true };
        }
        
        try {
            const params = {
                appid: this.apiKey,
                units: options.units || 'metric',
                lang: options.lang || 'en',
                ...query
            };
            
            const response = await axios.get(this.baseURL, { 
                params,
                timeout: 10000 
            });
            
            const weatherData = {
                ...response.data,
                cached: false,
                fetchedAt: new Date().toISOString()
            };
            
            // Cache the result
            cache.set(cacheKey, weatherData);
            
            return weatherData;
            
        } catch (error) {
            if (error.response) {
                throw new Error(`Weather API Error: ${error.response.data.message}`);
            } else {
                throw new Error(`Network Error: ${error.message}`);
            }
        }
    }
}

const weatherService = new WeatherService(process.env.OPENWEATHER_API_KEY);

// Routes
app.get('/weather/city/:cityName', async (req, res) => {
    try {
        const { cityName } = req.params;
        const { units, lang } = req.query;
        
        const weather = await weatherService.getWeather(
            { q: cityName },
            { units, lang }
        );
        
        res.json({
            success: true,
            data: weather
        });
        
    } catch (error) {
        res.status(400).json({
            success: false,
            error: error.message
        });
    }
});

app.get('/weather/coordinates', async (req, res) => {
    try {
        const { lat, lon, units, lang } = req.query;
        
        if (!lat || !lon) {
            return res.status(400).json({
                success: false,
                error: 'Latitude and longitude are required'
            });
        }
        
        const weather = await weatherService.getWeather(
            { lat: parseFloat(lat), lon: parseFloat(lon) },
            { units, lang }
        );
        
        res.json({
            success: true,
            data: weather
        });
        
    } catch (error) {
        res.status(400).json({
            success: false,
            error: error.message
        });
    }
});

app.listen(3000, () => {
    console.log('Weather API server running on port 3000');
});
```

**2. React Hook for Weather Data:**
```javascript
import { useState, useEffect, useCallback } from 'react';

export const useWeatherData = (location, options = {}) => {
    const [weather, setWeather] = useState(null);
    const [loading, setLoading] = useState(false);
    const [error, setError] = useState(null);
    
    const fetchWeather = useCallback(async () => {
        if (!location) return;
        
        setLoading(true);
        setError(null);
        
        try {
            const params = new URLSearchParams({
                appid: process.env.REACT_APP_OPENWEATHER_API_KEY,
                units: options.units || 'metric',
                lang: options.lang || 'en',
                ...location
            });
            
            const response = await fetch(
                `https://api.openweathermap.org/data/2.5/weather?${params}`
            );
            
            const data = await response.json();
            
            if (response.ok) {
                setWeather(data);
            } else {
                setError(data.message || 'Failed to fetch weather data');
            }
            
        } catch (err) {
            setError('Network error occurred');
        } finally {
            setLoading(false);
        }
    }, [location, options.units, options.lang]);
    
    useEffect(() => {
        fetchWeather();
    }, [fetchWeather]);
    
    return { weather, loading, error, refetch: fetchWeather };
};

// Usage in component
const WeatherComponent = ({ city }) => {
    const { weather, loading, error } = useWeatherData({ q: city });
    
    if (loading) return Loading weather...;
    if (error) return Error: {error};
    if (!weather) return null;
    
    return (
        
            {weather.name}
            {weather.main.temp}°C
            {weather.weather[0].description}
        
    );
};
```

### Mobile Development Examples

**1. React Native Implementation:**
```javascript
import React, { useState, useEffect } from 'react';
import { View, Text, ActivityIndicator, Alert } from 'react-native';
import * as Location from 'expo-location';

const WeatherScreen = () => {
    const [weather, setWeather] = useState(null);
    const [loading, setLoading] = useState(true);
    
    useEffect(() => {
        getLocationAndWeather();
    }, []);
    
    const getLocationAndWeather = async () => {
        try {
            // Request location permission
            const { status } = await Location.requestForegroundPermissionsAsync();
            if (status !== 'granted') {
                Alert.alert('Permission denied', 'Location access is required for weather data');
                return;
            }
            
            // Get current location
            const location = await Location.getCurrentPositionAsync({});
            const { latitude, longitude } = location.coords;
            
            // Fetch weather data
            const response = await fetch(
                `https://api.openweathermap.org/data/2.5/weather?lat=${latitude}&lon=${longitude}&appid=${API_KEY}&units=metric`
            );
            
            const weatherData = await response.json();
            
            if (response.ok) {
                setWeather(weatherData);
            } else {
                Alert.alert('Error', weatherData.message);
            }
            
        } catch (error) {
            Alert.alert('Error', 'Failed to get weather data');
        } finally {
            setLoading(false);
        }
    };
    
    if (loading) {
        return (
            
                
                Getting your weather...
            
        );
    }
    
    return (
        
            
                {weather.name}
            
            
                {Math.round(weather.main.temp)}°C
            
            
                {weather.weather[0].description}
            
            Feels like {Math.round(weather.main.feels_like)}°C
            Humidity: {weather.main.humidity}%
            Wind: {weather.wind.speed} m/s
        
    );
};
```

## Advanced Features

### Bulk Data Processing

For applications requiring weather data for multiple locations, implement efficient batch processing:

```python
import asyncio
import aiohttp
from typing import List, Dict

class BulkWeatherProcessor:
    def __init__(self, api_key: str, max_concurrent: int = 10):
        self.api_key = api_key
        self.semaphore = asyncio.Semaphore(max_concurrent)
        self.base_url = "https://api.openweathermap.org/data/2.5/weather"
    
    async def fetch_single_weather(self, session: aiohttp.ClientSession, location: Dict) -> Dict:
        async with self.semaphore:
            params = {
                'appid': self.api_key,
                'units': 'metric',
                **location
            }
            
            try:
                async with session.get(self.base_url, params=params) as response:
                    data = await response.json()
                    return {
                        'location': location,
                        'weather': data if response.status == 200 else None,
                        'error': None if response.status == 200 else data.get('message', 'Unknown error')
                    }
            except Exception as e:
                return {
                    'location': location,
                    'weather': None,
                    'error': str(e)
                }
    
    async def process_locations(self, locations: List[Dict]) -> List[Dict]:
        async with aiohttp.ClientSession() as session:
            tasks = [
                self.fetch_single_weather(session, location) 
                for location in locations
            ]
            return await asyncio.gather(*tasks)

# Usage
processor = BulkWeatherProcessor(API_KEY)
locations = [
    {'q': 'London,UK'},
    {'q': 'Paris,FR'},
    {'lat': 40.7128, 'lon': -74.0060},  # New York
    {'zip': '90210,US'}  # Beverly Hills
]

results = asyncio.run(processor.process_locations(locations))
```

### Real-time Weather Monitoring

Implement a monitoring system for critical weather conditions:

```python
import asyncio
import logging
from dataclasses import dataclass
from typing import Callable, List
from datetime import datetime

@dataclass
class WeatherAlert:
    location: str
    condition: str
    value: float
    threshold: float
    timestamp: datetime

class WeatherMonitor:
    def __init__(self, api_key: str):
        self.api_key = api_key
        self.alert_callbacks: List[Callable] = []
        self.monitoring = False
        
    def add_alert_callback(self, callback: Callable):
        self.alert_callbacks.append(callback)
    
    def check_conditions(self, weather_data: Dict, location: str) -> List[WeatherAlert]:
        alerts = []
        now = datetime.now()
        
        # Temperature alerts
        temp = weather_data['main']['temp']
        if temp > 35:  # Hot weather alert
            alerts.append(WeatherAlert(
                location=location,
                condition='high_temperature',
                value=temp,
                threshold=35,
                timestamp=now
            ))
        elif temp  15:
            alerts.append(WeatherAlert(
                location=location,
                condition='high_wind',
                value=weather_data['wind']['speed'],
                threshold=15,
                timestamp=now
            ))
        
        # Severe weather alerts
        weather_id = weather_data['weather'][0]['id']
        if 200 <= weather_id <= 299:  # Thunderstorms
            alerts.append(WeatherAlert(
                location=location,
                condition='thunderstorm',
                value=weather_id,
                threshold=200,
                timestamp=now
            ))
        
        return alerts
    
    async def monitor_location(self, location: Dict, interval: int = 300):
        while self.monitoring:
            try:
                # Fetch weather data (implementation similar to previous examples)
                weather_data = await self.fetch_weather(location)
                
                if weather_data:
                    alerts = self.check_conditions(weather_data, str(location))
                    
                    for alert in alerts:
                        for callback in self.alert_callbacks:
                            try:
                                callback(alert)
                            except Exception as e:
                                logging.error(f"Alert callback failed: {e}")
                
            except Exception as e:
                logging.error(f"Monitoring error for {location}: {e}")
            
            await asyncio.sleep(interval)
    
    def start_monitoring(self, locations: List[Dict], interval: int = 300):
        self.monitoring = True
        tasks = [
            self.monitor_location(location, interval) 
            for location in locations
        ]
        return asyncio.gather(*tasks)
    
    def stop_monitoring(self):
        self.monitoring = False

# Usage
def alert_handler(alert: WeatherAlert):
    print(f"WEATHER ALERT: {alert.condition} in {alert.location}")
    print(f"Value: {alert.value}, Threshold: {alert.threshold}")
    # Send to notification service, database, etc.

monitor = WeatherMonitor(API_KEY)
monitor.add_alert_callback(alert_handler)

locations = [
    {'q': 'Miami,FL,US'},
    {'q': 'Phoenix,AZ,US'},
    {'lat': 25.7617, 'lon': -80.1918}  # Miami coordinates
]

# Start monitoring (run in background)
asyncio.run(monitor.start_monitoring(locations, interval=600))  # Check every 10 minutes
```

## Troubleshooting

### Common Issues and Solutions

**1. Invalid API Key Errors:**
```
Error: {"cod":401,"message":"Invalid API key"}
```

**Solutions:**
- Verify API key is correctly copied from dashboard
- Check for extra spaces or special characters
- Ensure API key is active (newly created keys may take a few minutes)
- Confirm your account email is verified

**2. City Not Found Errors:**
```
Error: {"cod":"404","message":"city not found"}
```

**Solutions:**
- Try different city name formats: "London", "London,UK", "London,GB"
- Use city ID instead of name for ambiguous cities
- Check spelling and include country code for better accuracy
- Use coordinates for precise location matching

**3. Rate Limit Exceeded:**
```
Error: {"cod":429,"message":"Your account is temporary blocked"}
```

**Solutions:**
- Implement proper request throttling
- Add delays between requests
- Use caching to reduce API calls
- Consider upgrading to a paid plan

**4. Timeout Issues:**

```python
import requests
from requests.adapters import HTTPAdapter
from requests.packages.urllib3.util.retry import Retry

def create_robust_session():
    session = requests.Session()
    
    # Configure retries
    retry_strategy = Retry(
        total=3,
        status_forcelist=[429, 500, 502, 503, 504],
        method_whitelist=["HEAD", "GET", "OPTIONS"],
        backoff_factor=1
    )
    
    adapter = HTTPAdapter(max_retries=retry_strategy)
    session.mount("http://", adapter)
    session.mount("https://", adapter)
    
    return session

# Usage
session = create_robust_session()
response = session.get(url, params=params, timeout=10)
```

### Debugging Tools

**1. Request/Response Logging:**
```python
import logging
import requests

# Enable debug logging
logging.basicConfig()
logging.getLogger().setLevel(logging.DEBUG)
requests_log = logging.getLogger("requests.packages.urllib3")
requests_log.setLevel(logging.DEBUG)
requests_log.propagate = True

# Your API calls will now show detailed information
```

**2. Response Validation:**
```python
def debug_weather_response(response_data):
    print("=== Weather API Response Debug ===")
    print(f"Response type: {type(response_data)}")
    print(f"Response keys: {list(response_data.keys()) if isinstance(response_data, dict) else 'Not a dict'}")
    
    if isinstance(response_data, dict):
        if 'cod' in response_data:
            print(f"Status code: {response_data['cod']}")
        
        if 'message' in response_data:
            print(f"Message: {response_data['message']}")
        
        if 'main' in response_data:
            print(f"Temperature: {response_data['main'].get('temp', 'N/A')}")
            print(f"Humidity: {response_data['main'].get('humidity', 'N/A')}")
        
        if 'weather' in response_data and response_data['weather']:
            print(f"Weather: {response_data['weather'][0].get('description', 'N/A')}")
    
    print("===================================")
```

## Additional Resources

### Official Documentation and Tools

- **Main API Documentation:** [OpenWeatherMap Current Weather API]
- **City List Download:** [Bulk City List](http://bulk.openweathermap.org/sample/)
- **Weather Condition Codes:** [Weather Icons and Conditions](https://openweathermap.org/weather-conditions)
- **API Pricing Plans:** [Subscription Plans](https://openweathermap.org/price)
- **Terms of Service:** [Legal Terms](https://openweathermap.org/terms)

### Community Resources and Libraries

**Python Libraries:**
- `pyowm`: Official Python wrapper
- `openweathermapy`: Alternative Python client
- `weather-api`: Simplified weather client

**JavaScript/Node.js Libraries:**
- `openweather-apis`: Popular Node.js wrapper
- `weather-js`: Simple weather client
- `node-weather-api`: Full-featured client

**Mobile SDKs:**
- iOS: OpenWeatherMap iOS SDK
- Android: OpenWeatherMap Android SDK
- React Native: Community packages available

### Support and Community

- **Developer Forum:** [OpenWeatherMap Community](https://openweathermap.org/community)
- **Support Center:** [Help and Documentation](https://openweathermap.org/faq)
- **Status Page:** [Service Status](https://status.openweathermap.org/)