# esp8266-weather-WeatherApi

ESP8266/Arduino client for [WeatherAPI.com](https://www.weatherapi.com/) - a
single HTTPS request returns both current conditions and a multi-day
forecast. Get a free API key at https://www.weatherapi.com/signup.aspx (see
their [docs](https://www.weatherapi.com/docs/) for the full request/response
reference).

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
certificate validation, at the cost of not verifying the server's identity.
This avoids the maintenance burden of a pinned certificate fingerprint,
which breaks on every certificate renewal. If you need certificate
validation, use `client.setTrustAnchors()` instead, with WeatherAPI.com's
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


## License

This project is free software, released under the **GNU General Public License v3.0**. You may redistribute and/or modify it under those terms; see [LICENSE.md](LICENSE.md) for the full text.
