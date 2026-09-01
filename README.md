# esp8266-weather-WeatherApi

ESP8266/Arduino client for [WeatherAPI.com](https://www.weatherapi.com/) - a
single HTTPS request returns both current conditions and a multi-day
forecast. Get a free API key at https://www.weatherapi.com/signup.aspx (see
their [docs](https://www.weatherapi.com/docs/) for the full request/response
reference).

## Why this replaced the old HeWeather client

This library previously targeted `free-api.heweather.com` (the old HeWeather
v6 "s6" API). That domain no longer resolves at all - the service rebranded
to QWeather years ago and retired the old free API tier/domain along with it.
Two more problems made the old code worth replacing rather than patching:

- It pinned a TLS certificate fingerprint (`client.setFingerprint(...)`) that
  goes stale every time the server's certificate renews - a maintenance trap
  even setting the dead domain aside.
- An example API key was accidentally left in a source comment.

WeatherAPI.com's JSON response schema happens to closely match this
project's previous [esp8266-weather-APIXU](https://github.com/bobhuang1/esp8266-weather-APIXU)
library (WeatherAPI.com originated from the same team as the now-shut-down
APIXU service and kept its API contract), so the new `WeatherApiWeather`
class combines both current + forecast into one call, more efficient than
issuing two separate requests.

## Usage

```cpp
#include <ESP8266WiFi.h>
#include "WeatherApiWeather.h"

WeatherApiWeather weatherClient;
WeatherApiCurrentData current;
WeatherApiForecastData forecast[3]; // must match maxForecasts below

void setup() {
  // ... connect to WiFi first ...

  uint8_t forecastsReceived = weatherClient.updateWeather(
      &current, forecast,
      "YOUR_WEATHERAPI_COM_KEY",
      "London",   // or "lat,lon", a zip code, etc. - see WeatherAPI.com docs
      "en",       // language for condition text, e.g. "en", "zh"
      3);         // number of forecast days (check your plan's limit)

  Serial.printf("Now: %.1fC, %s\n", current.temp_c, current.text.c_str());
  for (uint8_t i = 0; i < forecastsReceived; i++) {
    Serial.printf("%s: %.1f/%.1fC, %s\n", forecast[i].date.c_str(),
                  forecast[i].mintemp_c, forecast[i].maxtemp_c, forecast[i].text.c_str());
  }
}
```

## Security note

The client uses `WiFiClientSecure::setInsecure()`, which skips TLS
certificate validation. This avoids the stale-fingerprint trap the previous
version fell into (a pinned fingerprint breaks on every certificate renewal),
at the cost of not verifying the server's identity. If you need certificate
validation, replace it with `client.setTrustAnchors()` using WeatherAPI.com's
current root CA certificate.

## Forecast fields

Note that WeatherAPI.com's daily forecast is a day-level aggregate, not the
day/night split some other providers offer: `WeatherApiForecastData` has one
`text`/`code` per day (not separate day and night condition text) and
`maxwind_kph` (a single peak wind speed for the day, no discrete direction -
wind direction is only available at hourly granularity, which this library
doesn't parse). `chanceOfRain` maps to WeatherAPI.com's `daily_chance_of_rain`.

## Dependencies

- [JsonStreamingParser](https://github.com/squix78/json-streaming-parser)
  (Arduino Library Manager)
- ESP8266 Arduino core (`ESP8266WiFi`, `ESP8266HTTPClient`, `WiFiClientSecure`)

## Icon mapping

`getMeteoconIcon()` maps WeatherAPI.com's numeric condition codes to a
two-character glyph pair (day-icon, night-icon) from the "Meteocons" icon
font used by several ESP8266 weather station projects. You'll need that font
(or your own icon set) to actually render these; this library only returns
the mapping.
