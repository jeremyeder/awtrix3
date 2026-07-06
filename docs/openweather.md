# OpenWeather

The native Weather app fetches current outdoor weather directly from OpenWeather on the ESP32. It does not require a companion process on another computer.

## Configuration

Configure the device from the AWTRIX web interface after flashing firmware with the native Weather app.

| Web field | Purpose |
|-----------|---------|
| `OpenWeather API Key` | OpenWeather API key. Stored locally on the clock in `/DoNotTouch.json`; it is not returned by `/api/settings`. |
| `OpenWeather Lat` | Latitude for the weather location. |
| `OpenWeather Lon` | Longitude for the weather location. |
| `OpenWeather Units` | Optional. Use `metric`, `imperial`, or `standard`. When empty, the clock follows the `CEL` setting. |
| `OpenWeather Interval` | Poll interval in seconds. The firmware enforces a minimum of 300 seconds. Default is 600 seconds. |

The Weather app itself is controlled by the Settings API key `WEA`. Set `WEA` to `true` and reboot or reload the native app loop to display it.

Example Settings API payload:

```json
{
  "WEA": true
}
```

## Behavior

The app calls OpenWeather's current weather endpoint with latitude, longitude, units, and API key. It displays an 8x8 weather icon plus the rounded current temperature on the 32x8 matrix.

Display states:

| Text | Meaning |
|------|---------|
| `SET` | API key or coordinates are missing. |
| `...` | Weather is configured and the first fetch is pending. |
| `E401` | OpenWeather rejected the API key. |
| `ENET` | The clock is not connected or could not start the request. |
| `EJSON` | OpenWeather returned a response the clock could not parse. |
| `EDATA` | The response did not contain temperature or humidity fields. |

## Secret Handling

Do not commit real OpenWeather keys or exact private coordinates. Store them only on the physical clock through the web interface or by uploading a local-only `/DoNotTouch.json` backup.
