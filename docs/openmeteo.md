# Open-Meteo

The native Weather app fetches current outdoor weather directly from Open-Meteo on the ESP32. It does not require an API key or a companion process on another computer.

## Configuration

Configure the device from the AWTRIX web interface after flashing firmware with the native Weather app.

| Web field | Purpose |
|-----------|---------|
| `Open-Meteo Lat` | Latitude for the weather location. |
| `Open-Meteo Lon` | Longitude for the weather location. |
| `Open-Meteo Units` | Optional. Use `celsius`, `fahrenheit`, `metric`, or `imperial`. When empty, the clock follows the `CEL` setting. |
| `Open-Meteo Interval` | Poll interval in seconds. The firmware enforces a minimum of 300 seconds. Default is 600 seconds. |

The Weather app itself is controlled by the Settings API key `WEA`. Set `WEA` to `true` and reboot or reload the native app loop to display it.

Example Settings API payload:

```json
{
  "WEA": true
}
```

## Behavior

The app calls Open-Meteo's forecast endpoint with latitude, longitude, current weather variables, and temperature unit. It displays an 8x8 weather icon plus the rounded current temperature on the 32x8 matrix.

Display states:

| Text | Meaning |
|------|---------|
| `SET` | Coordinates are missing. |
| `...` | Weather is configured and the first fetch is pending. |
| `E400` | Open-Meteo rejected the request parameters. |
| `ENET` | The clock is not connected or could not start the request. |
| `EJSON` | Open-Meteo returned a response the clock could not parse. |
| `EDATA` | The response did not contain temperature or humidity fields. |

## Privacy

No weather API key is required. Do not commit exact private coordinates; store them only on the physical clock through the web interface or by uploading a local-only `/DoNotTouch.json` backup.
