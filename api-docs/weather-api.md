# Weather API Documentation

## Table of Contents
- [Introduction](#introduction)
- [API Base URL](#api-base-url)
- [Authentication](#authentication)
- [Endpoints](#endpoints)
  - [Get Current Weather](#get-current-weather)
  - [Get Weather Forecast](#get-weather-forecast)
- [Rate Limiting](#rate-limiting)
- [Code Examples](#code-examples)
  - [Python](#python)
  - [JavaScript](#javascript)
  - [PHP](#php)
  - [Ruby](#ruby)
  - [Java](#java)
- [Official SDKs and Libraries](#official-sdks-and-libraries)
- [Versioning and Changelog](#versioning-and-changelog)
- [Security Best Practices](#security-best-practices)
- [Troubleshooting](#troubleshooting)
- [Support](#support)

## Introduction
This documentation describes the Weather API endpoints that allow you to retrieve current weather data and forecasts for locations worldwide.

## API Base URL
https://api.weatherapi.example.com/v1

## Authentication
All API requests require an API key passed in the header:

```http
X-API-Key: your_api_key_here
```

## Endpoints

### Get Current Weather
Returns current weather data for a specified location.

**Endpoint:** `/current`

**Method:** GET

**Parameters:**
| Parameter | Type   | Required | Description                    |
|-----------|--------|----------|--------------------------------|
| city      | string | Yes      | City name (e.g., "London")    |
| country   | string | No       | Country code (e.g., "UK")     |

**Response Format:**
```json
{
    "location": {
        "city": "London",
        "country": "UK",
        "lat": 51.52,
        "lon": -0.11
    },
    "current": {
        "temperature": 18,
        "humidity": 72,
        "condition": "Partly cloudy",
        "wind_speed": 12
    }
}
```

### Get Weather Forecast
Returns weather forecast data for the next 5 days.

**Endpoint:** `/forecast`

**Method:** GET

**Parameters:**
| Parameter | Type   | Required | Description                    |
|-----------|--------|----------|--------------------------------|
| city      | string | Yes      | City name (e.g., "London")    |
| country   | string | No       | Country code (e.g., "UK")     |
| days      | number | No       | Number of days (default: 5)    |

**Response Format:**
```json
{
    "location": {
        "city": "London",
        "country": "UK",
        "lat": 51.52,
        "lon": -0.11
    },
    "forecast": [
        {
            "date": "2024-03-19",
            "temperature": {
                "max": 20,
                "min": 12
            },
            "condition": "Sunny",
            "humidity": 65
        }
    ]
}
```

**Error Responses:**
| Status Code | Description                           |
|-------------|---------------------------------------|
| 400         | Bad Request - Invalid parameters      |
| 401         | Unauthorized - Invalid or missing API key |
| 404         | Not Found - City not found           |
| 429         | Too Many Requests - Rate limit exceeded |
| 500         | Internal Server Error                |

**Error Response Examples:**
```json
{
    "error": {
        "code": 400,
        "message": "Invalid city name provided"
    }
}
```

```json
{
    "error": {
        "code": 401,
        "message": "Invalid API key"
    }
}
```

```json
{
    "error": {
        "code": 429,
        "message": "Rate limit exceeded",
        "retry_after": 3600
    }
}
```

## Rate Limiting

The API implements rate limiting to ensure fair usage:

| Plan      | Rate Limit                |
|-----------|---------------------------|
| Free      | 60 requests per hour      |
| Basic     | 1000 requests per hour    |
| Premium   | 5000 requests per hour    |

When you exceed the rate limit, you'll receive a 429 error response with a `Retry-After` header indicating when you can resume making requests.

## Code Examples

### Python
```python
import requests

API_KEY = "your_api_key_here"
BASE_URL = "https://api.weatherapi.example.com/v1"

def get_current_weather(city, country=None):
    headers = {"X-API-Key": API_KEY}
    params = {"city": city, "country": country}
    
    response = requests.get(f"{BASE_URL}/current", headers=headers, params=params)
    return response.json()

# Example usage
weather = get_current_weather("London", "UK")
print(f"Temperature: {weather['current']['temperature']}°C")
```

### JavaScript
```javascript
const API_KEY = 'your_api_key_here';
const BASE_URL = 'https://api.weatherapi.example.com/v1';

async function getCurrentWeather(city, country) {
    const params = new URLSearchParams({ city, country });
    const response = await fetch(`${BASE_URL}/current?${params}`, {
        headers: {
            'X-API-Key': API_KEY
        }
    });
    return await response.json();
}

// Example usage
getCurrentWeather('London', 'UK')
    .then(weather => console.log(`Temperature: ${weather.current.temperature}°C`))
    .catch(error => console.error('Error:', error));
```

### PHP
```php
<?php
$apiKey = 'your_api_key_here';
$baseUrl = 'https://api.weatherapi.example.com/v1';

function getCurrentWeather($city, $country = null) {
    global $apiKey, $baseUrl;
    
    $params = http_build_query([
        'city' => $city,
        'country' => $country
    ]);
    
    $opts = [
        'http' => [
            'header' => "X-API-Key: $apiKey"
        ]
    ];
    
    $context = stream_context_create($opts);
    $response = file_get_contents("$baseUrl/current?$params", false, $context);
    return json_decode($response, true);
}

// Example usage
$weather = getCurrentWeather('London', 'UK');
echo "Temperature: {$weather['current']['temperature']}°C\n";
```

### Ruby
```ruby
require 'net/http'
require 'json'

API_KEY = 'your_api_key_here'
BASE_URL = 'https://api.weatherapi.example.com/v1'

def get_current_weather(city, country = nil)
  uri = URI("#{BASE_URL}/current")
  params = { city: city, country: country }
  uri.query = URI.encode_www_form(params)
  
  req = Net::HTTP::Get.new(uri)
  req['X-API-Key'] = API_KEY
  
  response = Net::HTTP.start(uri.hostname, uri.port, use_ssl: true) do |http|
    http.request(req)
  end
  
  JSON.parse(response.body)
end

# Example usage
weather = get_current_weather('London', 'UK')
puts "Temperature: #{weather['current']['temperature']}°C"
```

### Java
```java
import java.net.URI;
import java.net.http.*;
import java.util.HashMap;
import org.json.JSONObject;

public class WeatherAPI {
    private static final String API_KEY = "your_api_key_here";
    private static final String BASE_URL = "https://api.weatherapi.example.com/v1";
    
    public static JSONObject getCurrentWeather(String city, String country) throws Exception {
        String params = String.format("city=%s&country=%s", city, country);
        HttpClient client = HttpClient.newHttpClient();
        
        HttpRequest request = HttpRequest.newBuilder()
            .uri(URI.create(BASE_URL + "/current?" + params))
            .header("X-API-Key", API_KEY)
            .build();
            
        HttpResponse<String> response = client.send(request, 
            HttpResponse.BodyHandlers.ofString());
            
        return new JSONObject(response.body());
    }
    
    public static void main(String[] args) {
        try {
            JSONObject weather = getCurrentWeather("London", "UK");
            System.out.printf("Temperature: %d°C%n", 
                weather.getJSONObject("current").getInt("temperature"));
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

## Official SDKs and Libraries

The Weather API provides official SDK libraries for popular programming languages:

| Language   | Package Name          | Installation Command              |
|------------|----------------------|-----------------------------------|
| Python     | weatherapi-python    | `pip install weatherapi-python`   |
| JavaScript | weatherapi-js        | `npm install weatherapi-js`       |
| Java       | weatherapi-java      | `maven install weatherapi-java`   |
| Ruby       | weatherapi-ruby      | `gem install weatherapi-ruby`     |

Each SDK provides:
- Easy-to-use wrapper functions
- Type definitions
- Built-in error handling
- Automatic rate limiting
- Response caching options

## Versioning and Changelog

The Weather API uses semantic versioning (MAJOR.MINOR.PATCH). Current version: 1.0.0

### Latest Changes

#### v1.0.0 (2024-03-18)
- Initial public release
- Added current weather endpoint
- Added 5-day forecast endpoint
- Implemented rate limiting
- Released official SDKs

#### v0.9.0 (2024-03-01)
- Beta release
- Added authentication system
- Added error handling
- Added response caching

### Deprecation Policy
- APIs are versioned using path versioning (e.g., /v1/, /v2/)
- Old versions remain supported for 12 months after a new version release
- Deprecation notices are sent via email 3 months before end-of-life

## Security Best Practices

### API Key Management
- Never expose your API key in client-side code
- Rotate your API keys periodically
- Use different API keys for development and production
- Set up IP whitelisting in your dashboard

### HTTPS Requirements
- All API requests must use HTTPS
- HTTP requests will be rejected
- Keep your TLS/SSL certificates up to date

### Data Privacy
- Weather data is cached for 30 minutes
- No personal information is stored
- Location data is anonymized
- Usage logs are retained for 30 days

## Troubleshooting

### Common Issues and Solutions

#### 1. Authentication Errors (401)
- Verify your API key is valid and not expired
- Ensure the API key is correctly included in the X-API-Key header
- Check if your account is active and in good standing

#### 2. Rate Limiting (429)
- Check your current plan limits in the dashboard
- Implement exponential backoff in your requests
- Consider upgrading your plan if you consistently hit limits

#### 3. Invalid Parameters (400)
- Ensure city names are correctly spelled
- Use ISO country codes (e.g., "UK", "US")
- Verify the date format for forecast requests

## Support
- Visit our [Developer Forum](https://example.com/forum)
- Contact support: api-support@weatherapi.example.com
- Check our [Status Page](https://status.weatherapi.example.com)