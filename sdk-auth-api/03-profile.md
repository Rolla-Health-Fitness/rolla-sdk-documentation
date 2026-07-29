# Profile

This section covers setting up the user's profile from your app — and how doing so lets your users skip the SDK's account-details onboarding entirely. It also covers the [goal endpoints](#goal-endpoints): reading and changing the user's selected goals directly, including pre-selecting them so the SDK never has to ask.

**Set as much profile data as your app holds.** The required fields unlock the onboarding skip, and each optional field brings the profile a step closer to complete — the SDK feels like the user's own from the very first open.

Profile endpoints use `application/x-www-form-urlencoded` request bodies and require **both** the user's access token (`Authorization: Bearer`) — the same token you pass to `RollaConfiguration` — and your `Partner-ID` header, exactly like the authentication endpoints.

---

## Set Profile

```
POST /api/setprofile
```

Writes profile fields for the authenticated user. This is a **partial update**: send any subset of fields — omitted fields keep their current values. Call it after login and before first presenting the SDK to transfer profile data your app already holds.

Setting the profile isn't necessarily a one-time step: whenever the user edits their profile in your app, mirror the change with the same call — a single changed field is a valid request.

### Headers

| Header | Required | Description |
|--------|----------|-------------|
| `Authorization` | Yes | `Bearer <access_token>` |
| `Partner-ID` | Yes | Your partner identifier |
| `Content-Type` | Yes | `application/x-www-form-urlencoded` |

### Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `username` | string | No | Display name — first and last name go into this **one** field. Shown verbatim on leaderboards and the profile header; too-short or too-long values are rejected. |
| `birthdate` | string | No | `YYYY-MM-DD`; year between 1900 and the current year |
| `gender` | string | No | Exactly `male` or `female` |
| `height` | float | No | Height in cm, 100–255 |
| `weight` | float | No | Weight in kg, 40–300 (the SDK's own forms work in 40–180 — stay ≤ 180 for consistency) |
| `units` | string | No | `metric` or `imperial`. Unset ⇒ the SDK renders metric |
| `country` | string | No | ISO 3166-1 alpha-2 code (e.g. `GB`) |
| `city` | int | No | A **city ID** from [Countries and Cities](#countries-and-cities) — never free text |
| `language` | string | No | SDK UI language code (see the supported list below). **Strongly recommended** — see the note after the table |
| `timezone` | string | No | IANA timezone name (e.g. `Europe/Belgrade`). Unset ⇒ the SDK auto-syncs the device timezone |
| `max_heart_rate` | int | No | Max heart rate in bpm, or an empty string to have it determined automatically |

Send measurements in metric regardless of `units` — `units` only controls how the SDK displays them.

**Supported `language` codes** — the SDK ships these languages; send one of these exact codes:

| Language | Code |
|----------|------|
| English | `en` |
| German | `de` |
| Spanish | `es` |
| Croatian | `hr` |
| Bosnian | `bs` |
| Serbian (Latin) | `sr-Latn` |
| Serbian (Cyrillic) | `sr-Cyrl` |
| Arabic | `ar` |

> **Set `language` to match the SDK's UI language.** The Rolla backend generates localized content — goal labels, insights, and other server-produced text — from the profile's `language` field, independently of the language your app renders the SDK UI in. If they disagree (for example the SDK UI is Bosnian but the profile language is left at its default `en`), goals and insights come back in the profile's language while the rest of the UI is Bosnian. Set `language` here to the same language you configure the SDK with so everything the user sees is consistent. If you force the SDK language via `RollaConfiguration.language` (see the [iOS](../ios/05-configuration.md#language) / [Android](../android/05-configuration.md#language) configuration guides), the SDK writes it to the profile at startup for you.

### Example Request

```bash
curl -X POST "https://ross-rnd.rolla.cloud/api/setprofile" \
  -H "Authorization: Bearer YOUR_ACCESS_TOKEN" \
  -H "Partner-ID: your-partner-id" \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "username=Jane Runner&birthdate=1990-04-12&gender=female&height=170&weight=65&units=metric&country=GB"
```

### Example Response

```json
{
  "success": true
}
```

A rejected write returns `success: false` with a `reason`, e.g. `"Invalid birth date format"`, `"Birth year out of range"`, `"gender must be male or female"`, `"Height out of range"`, `"Weight out of range"`, `"units must be imperial or metric"`, `"unknown timezone"`, `"Unknown country code"`, `"Username too short"`, `"Username too long"`:

```json
{
  "success": false,
  "reason": "Height out of range"
}
```

> **Important:** `success: false` means the field was **not** stored — and if it was one of the fields required for the onboarding skip, the skip will not happen. Verify the response before presenting the SDK.

---

## Skipping SDK Onboarding

When a user first opens the SDK with an incomplete profile, the SDK shows its account-details onboarding form. You can skip it entirely by setting the profile fields in advance: on every SDK entry the SDK fetches the fresh profile and skips onboarding when **all** of these are present:

| Field | Condition |
|-------|-----------|
| `username` | Non-empty |
| `birthdate` | Set |
| `gender` | Set |
| `height` | > 0 |
| `weight` | > 0 — always required, even when `weight` is in `disabledModules`, because the SDK uses it to calculate calories |

Everything else (`units`, `country`, `city`, `language`, `timezone`, `max_heart_rate`) is optional — it never blocks the skip. The consent step and platform permission flows still run. The SDK's goals editor is also skipped — and a skipped user starts with **no goals selected** (there are no default goals). The SDK closes that gap at the first natural moment instead: see [Goal selection after the first data-source connect](#goal-selection-after-the-first-data-source-connect).

**Order matters:** await the `setprofile` success response **before** initializing and presenting the SDK. A write that lands too late means the onboarding form appears once, pre-filled.

### What happens, case by case

1. **You send all required fields** — e.g. `username`, `birthdate`, `gender`, `height`, `weight`. The user's first SDK open goes straight past onboarding; no account-details form, ever.
2. **You send a partial set** — e.g. `username`, `birthdate`, `gender`, but no `height`. The onboarding form appears **pre-filled** with everything you sent; the user only completes the gaps (here: height and weight).
3. **You send nothing** — the SDK runs its full built-in onboarding, exactly as before this feature.
4. **A write is rejected** (`success: false`) — the field is missing on the profile, so case 2 or 3 applies. Check responses.

### Verifying the skip

`GET /api/getprofile` and check the required fields are set — the next SDK open is then guaranteed to skip onboarding (the SDK evaluates the fresh profile on every entry). To deterministically test the onboarding form itself, use a fresh test user: `setprofile` is a partial update and cannot unset fields.

## Goal selection after the first data-source connect

Goals matter beyond the UI: the Rolla backend generates insights and parts of scoring from them, so a user without goals gets a reduced experience. Because the onboarding skip also skips the goals editor, the SDK asks such users to choose their goals **once** — right after their first successful data-source connect (band pairing or a health-platform connection), before landing on the Home screen. Saving the selection completes the step.

- **Users who already have goals never see it.** The step is skipped silently — no extra screens for anyone who chose goals at any point.
- **At most once per device.** Any completed goal selection — in this step, in the goals editor, or in an earlier session — permanently satisfies it on that device, including for a user who later deliberately deselects every goal. On a fresh install, a user who still has no goals selected is simply asked once more.
- **It never blocks on failures.** If goals can't be verified (offline, backend error), the user continues straight to Home.

To keep goals reachable from the Home screen afterwards, enable the Goals section (`showGoalsSection`) — see the [iOS](../ios/05-configuration.md#goals-on-home) / [Android](../android/05-configuration.md#goals-on-home) configuration guides. Your app can also pre-select goals server-side via the [goal endpoints](#goal-endpoints) below — a user with any enabled goal never sees this step.

---

## Get Profile

```
GET /api/getprofile
```

Returns the authenticated user's profile — use it to verify what you've set.

### Example Request

```bash
curl "https://ross-rnd.rolla.cloud/api/getprofile" \
  -H "Authorization: Bearer YOUR_ACCESS_TOKEN" \
  -H "Partner-ID: your-partner-id"
```

### Example Response

```json
{
  "success": true,
  "profile": {
    "user_name": "Jane Runner",
    "birthdate": "1990-04-12",
    "gender": "female",
    "height": 170,
    "weight": 65,
    "units": "metric",
    "country_code": "GB",
    "language": "en",
    "timezone": "Europe/Belgrade"
  }
}
```

---

## Goal Endpoints

> **Most integrations won't need these.** The SDK's goals UI covers the full experience on its own — selection, editing, and the one-time goal-selection step. The endpoints below are documented for completeness: reach for them only when your app wants direct control, such as showing goal state in its own UI or pre-selecting goals ahead of the user's first SDK open.

Goals are the user's selected focus areas (lose weight, improve sleep quality, …) — the Rolla backend generates insights and parts of scoring from them. These endpoints let your app read and change the selection directly; pre-selecting at least one goal means the [goal-selection step](#goal-selection-after-the-first-data-source-connect) never appears.

All three endpoints require the user's access token (`Authorization: Bearer`) **and** your `Partner-ID` header, exactly like the profile endpoints. The SDK's own goals UI lets users keep up to **5** goals selected at a time — stay within the same limit when pre-selecting.

### Get Goals

```
GET /goals/get
```

Returns the full goal catalog with each goal's enabled state. `name` and `description` arrive localized in the user's profile language; `icon` carries inline SVG markup.

#### Example Request

```bash
curl "https://ross-rnd.rolla.cloud/goals/get" \
  -H "Authorization: Bearer YOUR_ACCESS_TOKEN" \
  -H "Partner-ID: your-partner-id"
```

#### Example Response

```json
{
  "success": true,
  "goals": [
    {
      "id": 1,
      "icon": "<svg …>…</svg>",
      "name": "Lose weight",
      "description": "Reduce body weight safely and sustainably.",
      "enabled": true
    },
    {
      "id": 7,
      "icon": "<svg …>…</svg>",
      "name": "Improve sleep quality",
      "description": "Establish routines that promote deeper, restful sleep.",
      "enabled": false
    }
  ]
}
```

A fresh account starts with every goal `enabled: false` — no goals are pre-selected.

### Enable a Goal

```
POST /goals/enable
```

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `goal_id` | int | Yes | Goal `id` from `GET /goals/get` |

#### Example Request

```bash
curl -X POST "https://ross-rnd.rolla.cloud/goals/enable" \
  -H "Authorization: Bearer YOUR_ACCESS_TOKEN" \
  -H "Partner-ID: your-partner-id" \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "goal_id=1"
```

#### Example Response

```json
{"success": true}
```

### Disable a Goal

```
POST /goals/disable
```

Same parameters and headers as [Enable a Goal](#enable-a-goal); disabling a goal that isn't enabled is a harmless no-op.

```bash
curl -X POST "https://ross-rnd.rolla.cloud/goals/disable" \
  -H "Authorization: Bearer YOUR_ACCESS_TOKEN" \
  -H "Partner-ID: your-partner-id" \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "goal_id=1"
```

---

## Countries and Cities

The `city` parameter takes a numeric city ID, resolved via these two endpoints (both need the `Partner-ID` header):

```
GET /api/countries
GET /api/cities/{countryCode}
```

`/api/countries` returns `{ "success": true, "countries": [{ "code": "GB", "name": "United Kingdom" }, ...] }`; `/api/cities/GB` returns `{ "success": true, "cities": [{ "id": 1234, "city": "London", ... }, ...] }`. Match your city name against the list and send the `id`. If you can't resolve a city, simply omit it — `city` never blocks the onboarding skip.

---

**Previous:** [Authentication](02-authentication.md) | **Next:** [Error Handling](04-error-handling.md) | **Home:** [README](README.md)
