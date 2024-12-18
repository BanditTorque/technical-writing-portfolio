# Weather API Documentation

## Table of Contents
- [Introduction](#introduction)
- [API Base URL](#api-base-url)
- [Authentication](#authentication)
- [Endpoints](#endpoints)
  - [Get Current Weather](#get-current-weather)
  - [Get Weather Forecast](#get-weather-forecast)
- [Interactive API Examples](#interactive-api-examples)
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

## Interactive API Examples

You can test the Weather API endpoints directly from your browser using our interactive console:

### OpenAPI Specification
```yaml
openapi: 3.0.0
info:
  title: Weather API
  version: 1.0.0
  description: API for retrieving weather data and forecasts

servers:
  - url: https://api.weatherapi.example.com/v1

paths:
  /current:
    get:
      summary: Get Current Weather
      parameters:
        - name: city
          in: query
          required: true
          schema:
            type: string
          description: City name (e.g., "London")
        - name: country
          in: query
          required: false
          schema:
            type: string
          description: Country code (e.g., "UK")
        - name: X-API-Key
          in: header
          required: true
          schema:
            type: string
          description: Your API key
      responses:
        '200':
          description: Successful response
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/WeatherResponse'
        '400':
          $ref: '#/components/responses/BadRequest'
        '401':
          $ref: '#/components/responses/Unauthorized'

components:
  schemas:
    WeatherResponse:
      type: object
      properties:
        location:
          type: object
          properties:
            city:
              type: string
            country:
              type: string
            lat:
              type: number
            lon:
              type: number
        current:
          type: object
          properties:
            temperature:
              type: number
            humidity:
              type: number
            condition:
              type: string
            wind_speed:
              type: number
```

### Try It Out
You can test the API using our interactive console at [https://api.weatherapi.example.com/docs](https://api.weatherapi.example.com/docs) or using the embedded console below:

<div class="swagger-container">
  <!-- Note: In a real implementation, this would be replaced with an actual Swagger UI embed -->
  <iframe src="https://api.weatherapi.example.com/docs" width="100%" height="800px" frameborder="0"></iframe>
</div>

**Error Responses:**
| Status Code | Description                           |
|-------------|---------------------------------------|
| 400         | Bad Request - Invalid parameters      |
| 401         | Unauthorized - Invalid or missing API key |
| 404         | Not Found - City not found           |
| 429         | Too Many Requests - Rate limit exceeded |
| 500         | Internal Server Error                |

## Error Guide

### Common Error Scenarios

#### 1. Authentication Errors (401)
```json
{
    "error": {
        "code": 401,
        "message": "Invalid API key provided"
    }
}
```
**Common Causes:**
- Expired API key
- Malformed API key
- Unauthorized IP address
- Missing X-API-Key header

**Resolution Steps:**
1. Verify your API key in the dashboard
2. Check if your IP is whitelisted
3. Ensure the key is properly formatted in the header

#### 2. Rate Limit Errors (429)
```json
{
    "error": {
        "code": 429,
        "message": "Rate limit exceeded",
        "retry_after": 3600,
        "limit": "60 requests per hour"
    }
}
```
**Best Practices:**
```python
import time
import requests

def get_weather_with_retry(city, max_retries=3):
    for attempt in range(max_retries):
        try:
            response = requests.get(f"{BASE_URL}/current", 
                                 headers={"X-API-Key": API_KEY},
                                 params={"city": city})
            
            if response.status_code == 429:
                retry_after = int(response.headers.get('Retry-After', 60))
                time.sleep(retry_after)
                continue
                
            return response.json()
            
        except requests.exceptions.RequestException as e:
            if attempt == max_retries - 1:
                raise e
            time.sleep(2 ** attempt)  # Exponential backoff
```

#### 3. Invalid Parameters (400)
```json
{
    "error": {
        "code": 400,
        "message": "Invalid parameters",
        "details": {
            "city": "City name cannot be empty",
            "days": "Forecast days must be between 1 and 10"
        }
    }
}
```
**Common Issues:**
- Empty or invalid city names
- Invalid country codes
- Out of range forecast days
- Missing required parameters

**Example of Proper Parameters:**
```javascript
// Incorrect
fetch(`${BASE_URL}/forecast?city=&days=15`)

// Correct
fetch(`${BASE_URL}/forecast?city=London&country=UK&days=5`)
```

#### 4. Network Timeouts (408)
```json
{
    "error": {
        "code": 408,
        "message": "Request timeout"
    }
}
```
**Handling Timeouts:**
```javascript
async function getWeatherWithTimeout(city, timeout = 5000) {
    try {
        const controller = new AbortController();
        const timeoutId = setTimeout(() => controller.abort(), timeout);
        
        const response = await fetch(`${BASE_URL}/current?city=${city}`, {
            headers: { "X-API-Key": API_KEY },
            signal: controller.signal
        });
        
        clearTimeout(timeoutId);
        return await response.json();
    } catch (error) {
        if (error.name === 'AbortError') {
            throw new Error('Request timed out');
        }
        throw error;
    }
}
```

#### 5. Server Errors (500, 502, 503)
```json
{
    "error": {
        "code": 500,
        "message": "Internal server error",
        "request_id": "f7a8b934-1c63-4d12-bc84-f3c71e4516ae"
    }
}
```
**Fallback Strategy:**
```python
def get_weather_with_fallback(city):
    try:
        # Primary API endpoint
        response = requests.get(f"{BASE_URL}/current", 
                              headers={"X-API-Key": API_KEY},
                              params={"city": city})
        
        if response.status_code >= 500:
            # Fallback to cached data
            return get_cached_weather(city)
            
        return response.json()
        
    except requests.exceptions.RequestException:
        # Fallback to cached data
        return get_cached_weather(city)
```

### Error Handling Best Practices

1. **Always Check Status Codes**
```javascript
if (!response.ok) {
    const error = await response.json();
    throw new Error(`API Error: ${error.message}`);
}
```

2. **Implement Retry Logic**
```python
def retry_with_exponential_backoff(func, max_retries=3):
    for attempt in range(max_retries):
        try:
            return func()
        except Exception as e:
            if attempt == max_retries - 1:
                raise e
            time.sleep(2 ** attempt)
```

3. **Cache Successful Responses**
```javascript
const weatherCache = new Map();

async function getCachedWeather(city) {
    const cacheKey = `weather_${city}`;
    const cachedData = weatherCache.get(cacheKey);
    
    if (cachedData && Date.now() - cachedData.timestamp < 1800000) {
        return cachedData.data;
    }
    
    const data = await getCurrentWeather(city);
    weatherCache.set(cacheKey, {
        data,
        timestamp: Date.now()
    });
    
    return data;
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