# Profile

This section covers setting up the user's profile from your app — and how doing so lets your users skip the SDK's account-details onboarding entirely.

**Set as much profile data as your app holds.** The required fields unlock the onboarding skip, and each optional field brings the profile a step closer to complete — the SDK feels like the user's own from the very first open.

Profile endpoints use `application/x-www-form-urlencoded` request bodies and are authenticated with the user's access token (`Authorization: Bearer`) — the same token you pass to `RollaConfiguration`.

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
| `Content-Type` | Yes | `application/x-www-form-urlencoded` |

### Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `username` | string | No | Display name, 2–64 characters. First and last name go into this **one** field. Shown verbatim on leaderboards and the profile header. |
| `birthdate` | string | No | `YYYY-MM-DD`; year between 1900 and the current year |
| `gender` | string | No | Exactly `male` or `female` |
| `height` | float | No | Height in cm, 100–255 |
| `weight` | float | No | Weight in kg, 40–300 (the SDK's own forms work in 40–180 — stay ≤ 180 for consistency) |
| `units` | string | No | `metric` or `imperial`. Unset ⇒ the SDK renders metric |
| `country` | string | No | ISO 3166-1 alpha-2 code (e.g. `RS`) |
| `city` | int | No | A **city ID** from [Countries and Cities](#countries-and-cities) — never free text |
| `language` | string | No | 2-letter language code. Unset ⇒ the SDK resolves the device locale |
| `timezone` | string | No | IANA timezone name (e.g. `Europe/Belgrade`). Unset ⇒ the SDK auto-syncs the device timezone |
| `max_heart_rate` | int | No | Max heart rate in bpm, or an empty string to have it determined automatically |

Send measurements in metric regardless of `units` — `units` only controls how the SDK displays them.

### Example Request

```bash
curl -X POST "https://ross-rnd.rolla.cloud/api/setprofile" \
  -H "Authorization: Bearer YOUR_ACCESS_TOKEN" \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "username=Jane Runner&birthdate=1990-04-12&gender=female&height=170&weight=65&units=metric&country=RS"
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

> **Note:** The first time `gender` is set on a profile, Rolla's internal `registration` analytics event fires and the registration insight is triggered. Your app controls when a user counts as registered by choosing when to make this call.

---

## Skipping SDK Onboarding

When a user first opens the SDK with an incomplete profile, the SDK shows its account-details onboarding form. You can skip it entirely by setting the profile fields in advance: on every SDK entry the SDK fetches the fresh profile and skips onboarding when **all** of these are present:

| Field | Condition |
|-------|-----------|
| `username` | Non-empty |
| `birthdate` | Set |
| `gender` | Set |
| `height` | > 0 |
| `weight` | > 0 — **only required when your integration includes the weight module** (i.e. `weight` is not in `disabledModules`) |

Everything else (`units`, `country`, `city`, `language`, `timezone`, `max_heart_rate`) is optional — it never blocks the skip. The consent step and platform permission flows still run; the goals editor is also skipped, and users get default goals they can edit in-app.

**Order matters:** await the `setprofile` success response **before** initializing and presenting the SDK. A write that lands too late means the onboarding form appears once, pre-filled.

### What happens, case by case

1. **You send all required fields** — e.g. `username`, `birthdate`, `gender`, `height`, `weight`. The user's first SDK open goes straight past onboarding; no account-details form, ever.
2. **You send a partial set** — e.g. `username`, `birthdate`, `gender`, but no `height`. The onboarding form appears **pre-filled** with everything you sent; the user only completes the gaps (here: height and weight).
3. **You send nothing** — the SDK runs its full built-in onboarding, exactly as before this feature.
4. **A write is rejected** (`success: false`) — the field is missing on the profile, so case 2 or 3 applies. Check responses.

> **Important:** `username` and `gender` have **no edit UI inside the SDK** after onboarding. Send them correctly the first time — a wrong or missing name is what other users see on leaderboards.

### Verifying the skip

`GET /api/getprofile` and check the required fields are set — the next SDK open is then guaranteed to skip onboarding (the SDK evaluates the fresh profile on every entry). To deterministically test the onboarding form itself, use a fresh test user: `setprofile` is a partial update and cannot unset fields.

---

## Get Profile

```
GET /api/getprofile
```

Returns the authenticated user's profile — use it to verify what you've set.

### Example Request

```bash
curl "https://ross-rnd.rolla.cloud/api/getprofile" \
  -H "Authorization: Bearer YOUR_ACCESS_TOKEN"
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
    "country_code": "RS",
    "language": "en",
    "timezone": "Europe/Belgrade"
  }
}
```

---

## Countries and Cities

The `city` parameter takes a numeric city ID, resolved via these two endpoints (both `Authorization: Bearer`):

```
GET /api/countries
GET /api/cities/{countryCode}
```

`/api/countries` returns `{ "success": true, "countries": [{ "code": "RS", "name": "Serbia" }, ...] }`; `/api/cities/RS` returns `{ "success": true, "cities": [{ "id": 1234, "city": "Beograd", ... }, ...] }`. Match your city name against the list and send the `id`. If you can't resolve a city, simply omit it — `city` never blocks the onboarding skip.

---

**Prev:** [Authentication](02-authentication.md) | **Next:** [Error Handling](04-error-handling.md) | **Home:** [README](README.md)
