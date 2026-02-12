---
name: weather
description: Get current weather and forecasts (no API key required).
homepage: https://wttr.in/:help
metadata: { "openclaw": { "emoji": "🌤️", "requires": { "bins": ["curl"] } } }
---

# Weather

Two free services, no API keys needed.

## When to Use This Skill

**Use when:**

- Checking current weather for a location
- Getting weather forecasts (1-3 day)
- Need weather data in JSON format for scripting (Open-Meteo)
- Need weather as PNG image

**Don't use when:**

- Need historical weather data → use weather APIs with history support
- Need severe weather alerts → use dedicated alert services
- Need indoor climate data → use smart home sensors

**Success Criteria:**

- Weather data returned for correct location
- Format matches request (text, JSON, or PNG)

## wttr.in (primary)

Quick one-liner:

```bash
curl -s "wttr.in/London?format=3"
# Output: London: ⛅️ +8°C
```

Compact format:

```bash
curl -s "wttr.in/London?format=%l:+%c+%t+%h+%w"
# Output: London: ⛅️ +8°C 71% ↙5km/h
```

Full forecast:

```bash
curl -s "wttr.in/London?T"
```

Format codes: `%c` condition · `%t` temp · `%h` humidity · `%w` wind · `%l` location · `%m` moon

Tips:

- URL-encode spaces: `wttr.in/New+York`
- Airport codes: `wttr.in/JFK`
- Units: `?m` (metric) `?u` (USCS)
- Today only: `?1` · Current only: `?0`
- PNG: `curl -s "wttr.in/Berlin.png" -o /tmp/weather.png`

## Open-Meteo (fallback, JSON)

Free, no key, good for programmatic use:

```bash
curl -s "https://api.open-meteo.com/v1/forecast?latitude=51.5&longitude=-0.12&current_weather=true"
```

Find coordinates for a city, then query. Returns JSON with temp, windspeed, weathercode.

Docs: https://open-meteo.com/en/docs

### Common Pitfalls

**What NOT to do:**

- Forgetting to URL-encode city names with spaces: use `New+York` not `New York`
- Using wttr.in for programmatic JSON: use Open-Meteo instead (structured JSON)
- Not specifying units: defaults may not match user's preference (?m for metric, ?u for USCS)
