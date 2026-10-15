<div align="center">

# QASRA Geocoder

### Quran and Sunnah Research Academy, Ajmer

**Coordinates to place names · Place-name search · Simple JSON API**

[Try reverse lookup](https://geo.qasra.org/?lat=26.45298&lng=74.62349) · [Search for Ajmer](https://geo.qasra.org/search?q=Ajmer) · [Quick start](#quick-start) · [Rate limits](#rate-limits)

</div>

---

QASRA Geocoder helps your application turn a latitude and longitude into a nearby place name, district/state display and country. You can also search for places by the beginning of their names and use the returned coordinates in your application.

**Base URL:** `https://geo.qasra.org`

The API uses HTTP GET, returns JSON, supports browser CORS requests, and does not require an API key.

## Two ways to find a place

| What you already have | What to call | What you receive |
| --- | --- | --- |
| Latitude and longitude | `/?lat=26.45298&lng=74.62349` | A nearby place label, state/district display and country |
| The beginning of a place name | `/search?q=Ajmer` | Up to 10 matching names with their coordinates |

**Reverse geocoding** means coordinates → a place label. **Place-name search** means a name prefix → matching places and their coordinates. Search is a prefix lookup, not fuzzy matching or a full street-address search.

## Quick start

### Get a place from latitude and longitude

```bash
curl 'https://geo.qasra.org/?lat=26.45298&lng=74.62349'
```

Example response for these Ajmer coordinates:

```json
{
  "city": "Dargah Sharif",
  "state": "Ajmer, Rajasthan",
  "country": "India",
  "attribution": "Qasra Geocoding API - Free Public Service by Quran & Sunnah Research Academy, Ajmer"
}
```

This response example was observed from the public API on 16 October 2026. Place labels can change when the catalogue is updated.

### Search by name

```bash
curl 'https://geo.qasra.org/search?q=Ajmer'
```

The result is an array of up to 10 objects. For example, a matching item has this shape:

```json
{
  "name": "Ajmer",
  "state": "Rajasthan",
  "country": "India",
  "lat": 26.4521,
  "lng": 74.63867
}
```

An identical name can exist in more than one region or country. Display the returned state and country so users can choose the place they intended. The sample shows one item, not a guarantee that the entire response contains only one result. No matches return `[]`.

## Reverse lookup parameters

Use the root endpoint `/`:

| Parameter | Required | Description |
| --- | --- | --- |
| `lat` | Yes | Latitude in decimal degrees, between **-90 and 90** |
| `lng` | Yes | Longitude in decimal degrees, between **-180 and 180** |

Accepted alternatives are `latitude` for `lat`, and `longitude` or `lon` for `lng`. Use one spelling for each parameter in your application.

```text
/?lat=26.45298&lng=74.62349
/?latitude=26.45298&longitude=74.62349
/?latitude=26.45298&lon=74.62349
```

Latitude comes first in these examples. Longitude is east/west; latitude is north/south. Negative coordinates are valid: use negative latitude south of the equator and negative longitude west of Greenwich. Send numeric decimal degrees, without compass letters or degree symbols.

### Reverse response fields

| Field | Meaning |
| --- | --- |
| `city` | Resolved place label; it may name a locality, village, town or city |
| `state` | District/state display text where available; it can contain both, separated by a comma |
| `country` | Country display name, or the catalogue code when no display name is available |
| `attribution` | Provider credit returned by the service |

The response is a place label, not a postal address or a list of every nearby neighbourhood. It does not return street/house numbers, postal codes, location boundaries, distance, or a verified location-accuracy value. Very close coordinates can produce the same result; catalogue coverage varies by location. A result is not proof that a point lies within that place's administrative boundary.

## Place-name search parameters

Use `/search`:

| Parameter | Required for useful results | Description |
| --- | --- | --- |
| `q` | Yes | The beginning of the place name to look up |

Use `URLSearchParams` or URL encoding for names containing spaces or non-ASCII characters. An empty or whitespace-only search returns an empty array. `%` and `_` are not supported as user-supplied wildcard operators.

Search returns at most **10 results**. There is no pagination, country filter, autocomplete ranking guarantee or spelling-correction parameter. If several places match, make your prefix more specific and show the location context in your interface.

### Search response fields

| Field | Meaning |
| --- | --- |
| `name` | Matching place name |
| `state` | District/state display text where available |
| `country` | Country display name |
| `lat` | Matching place's numeric latitude |
| `lng` | Matching place's numeric longitude |

A user can select one search result and your application can use its `lat` and `lng` in a map, weather request or another location-based feature. Reverse lookup of those coordinates need not reproduce the identical label in every case.

## JavaScript: reverse lookup

Works with `fetch` in modern browsers and Node.js:

```javascript
async function reverseGeocode(latitude, longitude) {
  const url = new URL('https://geo.qasra.org/');
  url.searchParams.set('lat', String(latitude));
  url.searchParams.set('lng', String(longitude));

  const response = await fetch(url);
  const data = await response.json();
  if (!response.ok) {
    const error = new Error(data.error || 'Location lookup failed');
    error.status = response.status;
    error.retryAfter = response.headers.get('Retry-After');
    throw error;
  }
  return data;
}

const place = await reverseGeocode(26.45298, 74.62349);
console.log(place.city);
console.log(place.state);
console.log(place.country);
```

Use coordinates provided by your application's selected location or by the device's location permission flow. The API itself does not request GPS permission or discover the user's current location.

## JavaScript: place-name search

```javascript
async function searchPlaces(prefix) {
  const url = new URL('https://geo.qasra.org/search');
  url.searchParams.set('q', prefix.trim());

  const response = await fetch(url);
  const data = await response.json();
  if (!response.ok) throw new Error(data.error || 'Place search failed');
  return data;
}

const places = await searchPlaces('Ajmer');
for (const place of places) {
  console.log(place.name, place.state, place.country, place.lat, place.lng);
}
```

For a search input, wait until typing pauses or use a Search button. Do not call on every keypress. Handle `429` by respecting `Retry-After`; avoid an automatic retry loop.

## Python example

This example uses Python's standard library:

```python
import json
from urllib.parse import urlencode
from urllib.request import urlopen
from urllib.error import HTTPError

query = urlencode({"lat": 26.45298, "lng": 74.62349})
try:
    with urlopen("https://geo.qasra.org/?" + query, timeout=15) as response:
        place = json.load(response)
    print(place["city"], place["state"], place["country"])
except HTTPError as error:
    detail = json.loads(error.read().decode("utf-8"))
    print(error.code, detail.get("error"))
    if error.code in (429, 503):
        print("Retry after seconds:", error.headers.get("Retry-After"))
```

## Integration examples

### Show a location label in a weather or travel app

Obtain coordinates from the user's chosen location, call reverse lookup once, and show `city`, `state` and `country` next to the weather or map. Reuse that label while the selected location remains appropriate; do not repeatedly request it every second.

### Let someone choose their town

Send a name prefix to `/search`, show the matching names with state/country, and store the selected result's coordinates in your application's location settings. Similar names should remain separate choices rather than silently choosing the first item.

### Find coordinates before requesting Salah times

After the user selects a search result, pass its coordinates to QASRA Salah API:

```javascript
const selected = { lat: 26.4521, lng: 74.63867 };
const url = new URL('https://salah.qasra.org/');
url.searchParams.set('latitude', String(selected.lat));
url.searchParams.set('longitude', String(selected.lng));
url.searchParams.set('timezone', 'Asia/Kolkata');

const response = await fetch(url);
const data = await response.json();
if (!response.ok) throw new Error(data.error || 'Prayer lookup failed');
console.log(data.timings.hanafi.Fajr);
```

Choose the timezone appropriate to the selected place; this example uses an Indian location. Geocoder does not return a timezone. See the [QASRA Salah API guide](https://github.com/Qasraf/qasra-salah-api) for prayer and calendar parameters.

## Rate limits

Each public IP has its own Geocoder allowance, shared across reverse lookup and name search:

| Window | Requests per IP |
| --- | ---: |
| Minute | **60** |
| Hour | **300** |
| Day | **500** |

All limits apply together. Minute/hour windows are fixed. The daily allowance resets at **00:00 Asia/Kolkata (India time)**. `OPTIONS` preflight requests are exempt. Other accepted requests count even if the input is invalid or the API reuses a result. Devices sharing a public IP share the same Geocoder allowance. Calls from your backend share its outbound public IP allowance across its users.

QASRA Salah and Geocoder have separate allowances. Geocoder requests do not consume the Salah quota.

When a quota is exhausted, the API returns **HTTP 429** with `Retry-After`. Wait the stated number of seconds before trying again. That delay depends on whether a minute, hour or day allowance is exhausted; do not assume it is always 60 seconds.

A quota error contains `error`, `exceeded`, `retry_after_seconds` and `timezone`. The exhausted windows can be identified by `exceeded`, such as `minute`, `hour` or `day`. If the limiting service is unavailable, the API returns **503** with a retry delay.

For higher usage, contact [support@qasra.org](mailto:support@qasra.org). Contacting the academy does not automatically increase an allowance.

## HTTP responses and errors

| Status | Meaning | What your app should do |
| --- | --- | --- |
| `200` | Reverse lookup succeeded, or search returned an array | Read the JSON; a search array may be empty |
| `400` | Missing/invalid coordinates, out-of-range coordinates or missing client IP | Correct the request; do not repeat it unchanged |
| `404` | Reverse lookup found no nearby place | Let the user select or enter another location |
| `429` | One or more per-IP quotas exhausted | Respect `Retry-After` |
| `503` | Rate-limit service temporarily unavailable | Retry later using the supplied delay |
| `500` | Lookup or search failed unexpectedly | Handle gracefully; do not start a tight retry loop |

Errors have an `error` string. Check the HTTP status before treating the body as a successful place object or search array. CORS is enabled for browser clients, including access to `Retry-After` on rate-limit responses.

## Frequently asked questions

**Does the API know where the user is automatically?**

No. Supply latitude and longitude, or a name prefix. Your app handles device permission and location selection.

**Can I submit a complete street address?**

The search endpoint matches place-name prefixes. It is not a postal-address parser or house-number lookup.

**Why does a result contain a village or locality rather than a large city?**

The catalogue includes different kinds of populated places and local labels. The `city` field is the resolved label, not a guarantee of municipality type.

**Can I get all places in a district or every search match?**

There is no public district-list endpoint or paginated search. `/search` returns at most 10 matches.

**Will an identical name always mean the same place?**

No. Read the state, country and coordinates before selecting a match.

**Does changing a GPS decimal always produce a new place name?**

No. Nearby inputs can resolve to the same label. The API does not expose metre-level address precision.

**Can I call once and reuse the result?**

Yes. Reuse a suitable result while your selected location stays the same. Request again when needed rather than polling every second.

## Provider credit

The reverse response includes an `attribution` field naming the academy. Suggested display credit:

> Location information provided by Quran and Sunnah Research Academy, Ajmer (QASRA).

Link to [QASRA Geocoder](https://geo.qasra.org/). For usage questions or a higher allowance, contact [support@qasra.org](mailto:support@qasra.org).

---

<div align="center">

**QASRA Geocoder**  
A service by **Quran and Sunnah Research Academy, Ajmer**

</div>
