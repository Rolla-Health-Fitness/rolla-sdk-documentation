# Data Endpoints

All data endpoints require a valid Bearer token in the `Authorization` header and a `user_id` parameter identifying the user whose data you are accessing.

---

## Table of Contents

- [Activities](#activities)
- [Goals](#goals)
- [Health Data](#health-data)
  - [Heart Rate](#heart-rate)
  - [Heart Rate Variability (HRV)](#heart-rate-variability-hrv)
  - [Steps](#steps)
  - [Sleep](#sleep)
  - [Calories](#calories)
  - [Resting Heart Rate (RHR)](#resting-heart-rate-rhr)
  - [Move Hours](#move-hours)
  - [Active Points](#active-points)
  - [Health Score](#health-score)
- [Weight](#weight)
- [Blood Pressure](#blood-pressure)
- [Insights](#insights)
- [Device Info](#device-info)
- [Onboarding](#onboarding)
- [Parameter Reference](#parameter-reference)
- [Response Field Units](#response-field-units)

---

## Activities

### List Activities

```
POST /partners/v1/activities/list
```

Returns a paginated list of activities for a user.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `user_id` | string (UUID) | Yes | Partner's user identifier |
| `from` | string | No | Start date (`YYYY-MM-DD`) |
| `to` | string | No | End date (`YYYY-MM-DD`) |
| `limit` | integer | No | Results per page (1–100, default: 50) |
| `cursor` | string | No | Pagination cursor from previous response |

```bash
curl -X POST "https://ross.rolla.cloud/partners/v1/activities/list?limit=10&from=2024-11-01&to=2024-11-10" \
  -H "Authorization: Bearer YOUR_ACCESS_TOKEN" \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "user_id=0af73d3e-e092-49f4-b81b-7cf50e9c7782"
```

```json
{
  "success": true,
  "activities": [
    {
      "id": 123,
      "catalog_id": "running",
      "start_time": "2024-11-10 08:00:00",
      "duration": 3600,
      "distance": 5000
    }
  ],
  "next_cursor": "eyJpZCI6MTIzfQ"
}
```

Use `next_cursor` in the next request to retrieve the next page. When `next_cursor` is `null`, there are no more results.

---

### Add Activity

```
POST /partners/v1/activities/add
```

Creates or updates an activity. Idempotent — multiple POSTs with the same activity ID will update the existing record. Accepts `application/json` or `application/x-www-form-urlencoded` with JSON-encoded strings.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `user_id` | string (UUID) | Yes | Partner's user identifier |
| `summary` | object | Yes | Activity summary (see fields below) |
| `samples` | array | No | GPS/heart rate time-series samples |
| `artifacts` | object | No | Pace series, splits, HR series |

**Summary object fields:**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `id` | string (UUID) | Yes | Client-generated unique ID |
| `catalog_id` | string | Yes | Localized workout type identifier |
| `type` | string | Yes | One of: `walk`, `run`, `cardio`, `cycling` |
| `environment` | string | Yes | `outdoor` or `indoor` |
| `category` | string | No | Activity category (default: `cardio`) |
| `start_time` | integer | Yes | Start timestamp (epoch milliseconds UTC) |
| `end_time` | integer | Yes | End timestamp (epoch milliseconds UTC) |
| `total_distance_m` | number | No | Total distance in meters |
| `total_steps` | integer | No | Total step count |
| `total_calories` | number | No | Total calories burned |
| `total_active_points` | number | No | Activity points earned |
| `total_duration_s` | integer | No | Duration in seconds |
| `avg_hr_bpm` | integer | No | Average heart rate (1–300) |
| `max_hr_bpm` | integer | No | Maximum heart rate (1–300) |
| `avg_pace_s_per_km` | number | No | Average pace (seconds per km) |
| `avg_speed_mps` | number | No | Average speed (meters per second) |
| `source` | string | No | `rolla` (default) or `apple` |

```bash
curl -X POST "https://ross.rolla.cloud/partners/v1/activities/add" \
  -H "Authorization: Bearer YOUR_ACCESS_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "user_id": "0af73d3e-e092-49f4-b81b-7cf50e9c7782",
    "summary": {
      "id": "550e8400-e29b-41d4-a716-446655440000",
      "catalog_id": "workoutRunningOutdoor",
      "type": "run",
      "environment": "outdoor",
      "category": "cardio",
      "start_time": 1738689000000,
      "end_time": 1738690800000,
      "total_distance_m": 5012.3,
      "total_steps": 6240,
      "total_calories": 380.5,
      "total_active_points": 62.0,
      "total_duration_s": 1800,
      "avg_hr_bpm": 152,
      "max_hr_bpm": 171,
      "avg_pace_s_per_km": 358.0,
      "avg_speed_mps": 2.79
    },
    "samples": [
      {"ts": 1738689000000, "lat": 45.81, "lon": 15.98, "hr": 95}
    ],
    "artifacts": {
      "pace_series": [{"ts": 1738689060000, "pace_sec_per_km": 360.0}],
      "splits": [{"idx": 1, "distance_m": 1000.0, "duration_s": 360}],
      "hr_series": [{"ts": 1738689000000, "hr": 95}]
    }
  }'
```

```json
{
  "success": true
}
```

---

### Get Single Activity

```
POST /partners/v1/activities/get/{id}
```

Returns a single activity by ID.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `user_id` | string (UUID) | Yes | Partner's user identifier |
| `include` | string (query) | No | Comma-separated: `summary` (default), `artifacts`, `samples` |

```bash
curl -X POST "https://ross.rolla.cloud/partners/v1/activities/get/550e8400-e29b-41d4-a716-446655440000?include=summary,artifacts,samples" \
  -H "Authorization: Bearer YOUR_ACCESS_TOKEN" \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "user_id=0af73d3e-e092-49f4-b81b-7cf50e9c7782"
```

```json
{
  "success": true,
  "activity": {
    "summary": {
      "id": "550e8400-e29b-41d4-a716-446655440000",
      "catalog_id": "workoutRunningOutdoor",
      "type": "run",
      "environment": "outdoor",
      "start_time": 1738689000000,
      "end_time": 1738690800000,
      "total_distance_m": 5012.30,
      "total_calories": 380.50,
      "total_duration_s": 1800
    },
    "artifacts": { "..." : "..." },
    "samples": [ "..." ]
  }
}
```

---

### Quick List

```
POST /partners/v1/activities/quick_list
```

Returns a list of recent activity types (`catalog_id`) based on user activity and partner preferences. Returns 6 results.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `user_id` | string (UUID) | Yes | Partner's user identifier |

```bash
curl -X POST "https://ross.rolla.cloud/partners/v1/activities/quick_list" \
  -H "Authorization: Bearer YOUR_ACCESS_TOKEN" \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "user_id=0af73d3e-e092-49f4-b81b-7cf50e9c7782"
```

---

### Delete Activity

```
POST /partners/v1/activities/delete/{id}
```

Permanently removes an activity and its associated samples and artifacts. Synced biometric data (HR, calories, steps) is preserved.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `user_id` | string (UUID) | Yes | Partner's user identifier |

```bash
curl -X POST "https://ross.rolla.cloud/partners/v1/activities/delete/550e8400-e29b-41d4-a716-446655440000" \
  -H "Authorization: Bearer YOUR_ACCESS_TOKEN" \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "user_id=0af73d3e-e092-49f4-b81b-7cf50e9c7782"
```

```json
{
  "success": true
}
```

---

## Goals

### Get Goals

```
POST /partners/v1/goals/get
```

Returns available goals for the user. Goal name and description are localized according to the user's profile language.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `user_id` | string (UUID) | Yes | Partner's user identifier |

```bash
curl -X POST "https://ross.rolla.cloud/partners/v1/goals/get" \
  -H "Authorization: Bearer YOUR_ACCESS_TOKEN" \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "user_id=0af73d3e-e092-49f4-b81b-7cf50e9c7782"
```

```json
{
  "success": true,
  "goals": [
    {
      "id": 1,
      "icon": "<svg .../>",
      "name": "Lose weight",
      "description": "Reduce body weight safely and sustainably.",
      "enabled": false
    }
  ]
}
```

---

### Enable Goal

```
POST /partners/v1/goals/enable
```

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `user_id` | string (UUID) | Yes | Partner's user identifier |
| `goal_id` | integer | Yes | ID of the goal to enable |

```bash
curl -X POST "https://ross.rolla.cloud/partners/v1/goals/enable" \
  -H "Authorization: Bearer YOUR_ACCESS_TOKEN" \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "user_id=0af73d3e-e092-49f4-b81b-7cf50e9c7782&goal_id=1"
```

```json
{
  "success": true
}
```

---

### Disable Goal

```
POST /partners/v1/goals/disable
```

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `user_id` | string (UUID) | Yes | Partner's user identifier |
| `goal_id` | integer | Yes | ID of the goal to disable |

```bash
curl -X POST "https://ross.rolla.cloud/partners/v1/goals/disable" \
  -H "Authorization: Bearer YOUR_ACCESS_TOKEN" \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "user_id=0af73d3e-e092-49f4-b81b-7cf50e9c7782&goal_id=1"
```

```json
{
  "success": true
}
```

---

## Health Data

All health data endpoints follow a common pattern:

```
POST /partners/v1/health/{type}/get
```

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `user_id` | string (UUID) | Yes | Partner's user identifier |
| `from` | string | Yes | Start date (`YYYY-MM-DD`) |
| `to` | string | Yes | End date (`YYYY-MM-DD`) |
| `type` | string | Yes | Aggregation granularity (see [Type Parameter Values](#type-parameter-values)) |

---

### Heart Rate

```
POST /partners/v1/health/heartrate/get
```

**Type values:** `10_min`, `daily`, `monthly`

```bash
curl -X POST "https://ross.rolla.cloud/partners/v1/health/heartrate/get" \
  -H "Authorization: Bearer YOUR_ACCESS_TOKEN" \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "user_id=0af73d3e-e092-49f4-b81b-7cf50e9c7782&from=2024-11-10&to=2024-11-10&type=10_min"
```

```json
{
  "success": true,
  "heart_rate_data": [
    {
      "period_start": "2024-11-10 00:00:00",
      "min": 60,
      "max": 75,
      "avg": 68.5
    },
    {
      "period_start": "2024-11-10 00:10:00",
      "min": 65,
      "max": 80,
      "avg": 72.3
    }
  ]
}
```

**Add Heart Rate Data:**

```
POST /partners/v1/health/heartrate/add
```

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `user_id` | string (UUID) | Yes | Partner's user identifier |
| `heart_rate_data` | JSON array | Yes | Array of `{"timestamp": <epoch_sec>, "hr": <int>}` |
| `source` | string | No | `rolla` or `apple` |
| `synced_up_to` | integer | No | Sync checkpoint for non-rolla sources |

```bash
curl -X POST "https://ross.rolla.cloud/partners/v1/health/heartrate/add" \
  -H "Authorization: Bearer YOUR_ACCESS_TOKEN" \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "user_id=0af73d3e-e092-49f4-b81b-7cf50e9c7782&heart_rate_data=[{\"timestamp\":1755043140,\"hr\":60},{\"timestamp\":1755043150,\"hr\":65}]&source=apple"
```

```json
{
  "success": true
}
```

---

### Heart Rate Variability (HRV)

```
POST /partners/v1/health/hrv/get
```

**Type values:** `10_min`, `daily`, `monthly`

```bash
curl -X POST "https://ross.rolla.cloud/partners/v1/health/hrv/get" \
  -H "Authorization: Bearer YOUR_ACCESS_TOKEN" \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "user_id=0af73d3e-e092-49f4-b81b-7cf50e9c7782&from=2024-11-10&to=2024-11-10&type=daily"
```

```json
{
  "success": true,
  "hrv_data": [
    {
      "period_start": "2024-11-10",
      "min": 30,
      "max": 50,
      "avg": 40.2
    }
  ]
}
```

**Add HRV Data:**

```
POST /partners/v1/health/hrv/add
```

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `user_id` | string (UUID) | Yes | Partner's user identifier |
| `hrv_data` | JSON array | Yes | Array of `{"timestamp": <epoch_sec>, "hrv": <int>}` |
| `source` | string | No | `rolla` or `apple` |
| `synced_up_to` | integer | No | Sync checkpoint for non-rolla sources |

```bash
curl -X POST "https://ross.rolla.cloud/partners/v1/health/hrv/add" \
  -H "Authorization: Bearer YOUR_ACCESS_TOKEN" \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "user_id=0af73d3e-e092-49f4-b81b-7cf50e9c7782&hrv_data=[{\"timestamp\":1755043140,\"hrv\":60},{\"timestamp\":1755043150,\"hrv\":65}]&source=apple"
```

```json
{
  "success": true
}
```

---

### Steps

```
POST /partners/v1/health/steps/get
```

**Type values:** `hourly`, `daily`, `monthly`

```bash
curl -X POST "https://ross.rolla.cloud/partners/v1/health/steps/get" \
  -H "Authorization: Bearer YOUR_ACCESS_TOKEN" \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "user_id=0af73d3e-e092-49f4-b81b-7cf50e9c7782&from=2024-11-10&to=2024-11-10&type=daily"
```

```json
{
  "success": true,
  "steps": [
    {
      "period_start": "2024-11-10",
      "steps": 8500
    }
  ]
}
```

**Add Step/Calorie Data:**

```
POST /partners/v1/health/steps/add
```

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `user_id` | string (UUID) | Yes | Partner's user identifier |
| `steps` | JSON array | Yes | Array of `{"timestamp": <epoch_sec>, "steps_delta": <int>, "calories_delta": <int>}` |
| `source` | string | No | `rolla` or `apple` |
| `synced_up_to` | integer | No | Sync checkpoint for non-rolla sources |

```bash
curl -X POST "https://ross.rolla.cloud/partners/v1/health/steps/add" \
  -H "Authorization: Bearer YOUR_ACCESS_TOKEN" \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "user_id=0af73d3e-e092-49f4-b81b-7cf50e9c7782&steps=[{\"timestamp\":1755043140,\"steps_delta\":60,\"calories_delta\":100}]&source=apple"
```

```json
{
  "success": true
}
```

---

### Sleep

```
POST /partners/v1/health/sleep/get
```

**Type values:** `all` (individual sessions), `daily`, `monthly`

```bash
curl -X POST "https://ross.rolla.cloud/partners/v1/health/sleep/get" \
  -H "Authorization: Bearer YOUR_ACCESS_TOKEN" \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "user_id=0af73d3e-e092-49f4-b81b-7cf50e9c7782&from=2024-11-10&to=2024-11-10&type=daily"
```

```json
{
  "success": true,
  "sleep": [
    {
      "sleep_date": "2024-11-10",
      "total_sleep_minutes": 420
    }
  ]
}
```

**Add Sleep Data:**

```
POST /partners/v1/health/sleep/add
```

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `user_id` | string (UUID) | Yes | Partner's user identifier |
| `sleep_data` | JSON array | Yes | Array of `{"start_time": <epoch_sec>, "end_time": <epoch_sec>, "stage": "<stage>"}`. Stage: `awake`, `light`, `deep`, `rem` |
| `source` | string | No | `rolla` or `apple` |
| `synced_up_to` | integer | No | Sync checkpoint for non-rolla sources |

```bash
curl -X POST "https://ross.rolla.cloud/partners/v1/health/sleep/add" \
  -H "Authorization: Bearer YOUR_ACCESS_TOKEN" \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "user_id=0af73d3e-e092-49f4-b81b-7cf50e9c7782&sleep_data=[{\"start_time\":1755043000,\"end_time\":1755046600,\"stage\":\"light\"},{\"start_time\":1755046600,\"end_time\":1755050000,\"stage\":\"deep\"}]&source=apple"
```

```json
{
  "success": true
}
```

---

### Calories

```
POST /partners/v1/health/calories/get
```

**Type values:** `hourly`, `daily`, `monthly`

```bash
curl -X POST "https://ross.rolla.cloud/partners/v1/health/calories/get" \
  -H "Authorization: Bearer YOUR_ACCESS_TOKEN" \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "user_id=0af73d3e-e092-49f4-b81b-7cf50e9c7782&from=2024-11-10&to=2024-11-10&type=daily"
```

```json
{
  "success": true,
  "calories": [
    {
      "period_start": "2024-11-10",
      "calories": 2200
    }
  ]
}
```

---

### Resting Heart Rate (RHR)

```
POST /partners/v1/health/rhr/get
```

**Type values:** `daily`, `monthly`

```bash
curl -X POST "https://ross.rolla.cloud/partners/v1/health/rhr/get" \
  -H "Authorization: Bearer YOUR_ACCESS_TOKEN" \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "user_id=0af73d3e-e092-49f4-b81b-7cf50e9c7782&from=2024-11-10&to=2024-11-10&type=daily"
```

```json
{
  "success": true,
  "rhr": [
    {
      "period_start": "2024-11-10",
      "rhr": 58
    }
  ]
}
```

---

### Move Hours

```
POST /partners/v1/health/move_hours/get
```

**Type values:** `hourly`, `daily`, `monthly`

```bash
curl -X POST "https://ross.rolla.cloud/partners/v1/health/move_hours/get" \
  -H "Authorization: Bearer YOUR_ACCESS_TOKEN" \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "user_id=0af73d3e-e092-49f4-b81b-7cf50e9c7782&from=2024-11-10&to=2024-11-10&type=daily"
```

```json
{
  "success": true,
  "move_hours": [
    {
      "period_start": "2024-11-10",
      "move_hours": 2
    }
  ]
}
```

---

### Active Points

```
POST /partners/v1/health/active_points/get
```

**Type values:** `hourly`, `daily`, `monthly`

```bash
curl -X POST "https://ross.rolla.cloud/partners/v1/health/active_points/get" \
  -H "Authorization: Bearer YOUR_ACCESS_TOKEN" \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "user_id=0af73d3e-e092-49f4-b81b-7cf50e9c7782&from=2024-11-10&to=2024-11-10&type=daily"
```

```json
{
  "success": true,
  "active_points": [
    {
      "period_start": "2024-11-10",
      "active_points": 45
    }
  ]
}
```

---

### Health Score

```
POST /partners/v1/health/score/get
```

**Type values:** `daily`, `monthly`

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `user_id` | string (UUID) | Yes | Partner's user identifier |
| `from` | string | Yes | Start date (`YYYY-MM-DD`) |
| `to` | string | Yes | End date (`YYYY-MM-DD`) |
| `score` | string | Yes | Score type: `health`, `activity`, `readiness`, `steps`, `move_hours`, `active_points`, `sleep`, `rhr`, `hrv` |
| `type` | string | Yes | Aggregation: `daily`, `monthly` |

```bash
curl -X POST "https://ross.rolla.cloud/partners/v1/health/score/get" \
  -H "Authorization: Bearer YOUR_ACCESS_TOKEN" \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "user_id=0af73d3e-e092-49f4-b81b-7cf50e9c7782&from=2024-11-10&to=2024-11-10&score=health&type=daily"
```

```json
{
  "success": true,
  "score": [
    {
      "period_start": "2024-11-10",
      "score": 85
    }
  ]
}
```

---

## Weight

Full CRUD operations for weight entries, plus targets and aggregates.

### Add Weight Entry

```
POST /partners/v1/health/weight/add
```

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `user_id` | string (UUID) | Yes | Partner's user identifier |
| `weight` | float | Yes | Weight in kg (0.1–1000) |
| `timestamp` | integer | Yes | Epoch milliseconds |
| `source` | string | No | `rolla`, `apple`, `garmin`, `whoop`, `oura` |

```bash
curl -X POST "https://ross.rolla.cloud/partners/v1/health/weight/add" \
  -H "Authorization: Bearer YOUR_ACCESS_TOKEN" \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "user_id=0af73d3e-e092-49f4-b81b-7cf50e9c7782&weight=75.5&timestamp=1755043140000&source=apple"
```

```json
{
  "success": true
}
```

---

### Get Latest Weight

```
POST /partners/v1/health/weight/latest
```

Returns the latest weight entry, recent change, and current target.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `user_id` | string (UUID) | Yes | Partner's user identifier |

```bash
curl -X POST "https://ross.rolla.cloud/partners/v1/health/weight/latest" \
  -H "Authorization: Bearer YOUR_ACCESS_TOKEN" \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "user_id=0af73d3e-e092-49f4-b81b-7cf50e9c7782"
```

```json
{
  "success": true,
  "latest": { "..." : "..." },
  "change": { "..." : "..." },
  "target": { "..." : "..." }
}
```

---

### Get Weight Aggregates

```
POST /partners/v1/health/weight/get
```

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `user_id` | string (UUID) | Yes | Partner's user identifier |
| `from` | string | Yes | Start date (`YYYY-MM-DD`) |
| `to` | string | Yes | End date (`YYYY-MM-DD`) |
| `granularity` | string | No | `daily` (default), `weekly`, `monthly`, `yearly` |

```bash
curl -X POST "https://ross.rolla.cloud/partners/v1/health/weight/get" \
  -H "Authorization: Bearer YOUR_ACCESS_TOKEN" \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "user_id=0af73d3e-e092-49f4-b81b-7cf50e9c7782&from=2024-11-01&to=2024-11-30&granularity=daily"
```

```json
{
  "success": true,
  "aggregates": [ "..." ],
  "min_entry": { "..." : "..." },
  "max_entry": { "..." : "..." },
  "range_change": -1.5
}
```

---

### Get Weight History

```
POST /partners/v1/health/weight/history
```

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `user_id` | string (UUID) | Yes | Partner's user identifier |
| `from` | string | Yes | Start date (`YYYY-MM-DD`) |
| `to` | string | Yes | End date (`YYYY-MM-DD`) |
| `limit` | integer | No | Max entries (1–1000) |

```bash
curl -X POST "https://ross.rolla.cloud/partners/v1/health/weight/history" \
  -H "Authorization: Bearer YOUR_ACCESS_TOKEN" \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "user_id=0af73d3e-e092-49f4-b81b-7cf50e9c7782&from=2024-11-01&to=2024-11-30&limit=100"
```

```json
{
  "success": true,
  "history": [ "..." ],
  "min_entry": { "..." : "..." },
  "max_entry": { "..." : "..." },
  "range_change": -1.5
}
```

---

### Update Weight Entry

```
POST /partners/v1/health/weight/update
```

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `user_id` | string (UUID) | Yes | Partner's user identifier |
| `id` | integer | Yes | Weight entry ID |
| `weight` | float | Yes | Weight in kg |
| `timestamp` | integer | Yes | Epoch milliseconds |

```bash
curl -X POST "https://ross.rolla.cloud/partners/v1/health/weight/update" \
  -H "Authorization: Bearer YOUR_ACCESS_TOKEN" \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "user_id=0af73d3e-e092-49f4-b81b-7cf50e9c7782&id=42&weight=74.0&timestamp=1755043140000"
```

```json
{
  "success": true
}
```

---

### Delete Weight Entry

```
POST /partners/v1/health/weight/delete
```

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `user_id` | string (UUID) | Yes | Partner's user identifier |
| `id` | integer | Yes | Weight entry ID |

```bash
curl -X POST "https://ross.rolla.cloud/partners/v1/health/weight/delete" \
  -H "Authorization: Bearer YOUR_ACCESS_TOKEN" \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "user_id=0af73d3e-e092-49f4-b81b-7cf50e9c7782&id=42"
```

```json
{
  "success": true
}
```

---

### Get Weight Target

```
GET /partners/v1/health/weight/target
```

```bash
curl -X GET "https://ross.rolla.cloud/partners/v1/health/weight/target" \
  -H "Authorization: Bearer YOUR_ACCESS_TOKEN"
```

```json
{
  "success": true,
  "target": { "..." : "..." }
}
```

---

### Set Weight Target

```
POST /partners/v1/health/weight/target
```

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `user_id` | string (UUID) | Yes | Partner's user identifier |
| `weight` | float | Yes | Target weight in kg |

```bash
curl -X POST "https://ross.rolla.cloud/partners/v1/health/weight/target" \
  -H "Authorization: Bearer YOUR_ACCESS_TOKEN" \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "user_id=0af73d3e-e092-49f4-b81b-7cf50e9c7782&weight=70.0"
```

```json
{
  "success": true
}
```

---

### Delete Weight Target

```
POST /partners/v1/health/weight/target_delete
```

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `user_id` | string (UUID) | Yes | Partner's user identifier |

```bash
curl -X POST "https://ross.rolla.cloud/partners/v1/health/weight/target_delete" \
  -H "Authorization: Bearer YOUR_ACCESS_TOKEN" \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "user_id=0af73d3e-e092-49f4-b81b-7cf50e9c7782"
```

```json
{
  "success": true
}
```

---

## Blood Pressure

Full CRUD operations with ESC/ESH classification and aggregates.

### Add Blood Pressure Reading

```
POST /partners/v1/health/blood_pressure/add
```

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `user_id` | string (UUID) | Yes | Partner's user identifier |
| `systolic` | integer | Yes | Systolic pressure in mmHg (70–250) |
| `diastolic` | integer | Yes | Diastolic pressure in mmHg (40–150) |
| `timestamp` | integer | Yes | Epoch milliseconds |
| `note` | string | No | Optional note (max 500 characters) |
| `source` | string | No | `rolla`, `apple`, `garmin`, `whoop`, `oura` |
| `synced_up_to` | integer | No | Sync checkpoint for non-rolla sources |

Systolic must be greater than diastolic.

```bash
curl -X POST "https://ross.rolla.cloud/partners/v1/health/blood_pressure/add" \
  -H "Authorization: Bearer YOUR_ACCESS_TOKEN" \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "user_id=0af73d3e-e092-49f4-b81b-7cf50e9c7782&systolic=120&diastolic=80&timestamp=1755043140000&source=apple"
```

```json
{
  "success": true
}
```

---

### Get Latest Blood Pressure

```
POST /partners/v1/health/blood_pressure/latest
```

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `user_id` | string (UUID) | Yes | Partner's user identifier |

```bash
curl -X POST "https://ross.rolla.cloud/partners/v1/health/blood_pressure/latest" \
  -H "Authorization: Bearer YOUR_ACCESS_TOKEN" \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "user_id=0af73d3e-e092-49f4-b81b-7cf50e9c7782"
```

```json
{
  "success": true,
  "latest": {
    "id": 42,
    "systolic_mm_hg": 120,
    "diastolic_mm_hg": 80,
    "timestamp_tz": "2024-11-10 08:00:00+00",
    "note": null,
    "source": "rolla",
    "classification": "Optimal"
  }
}
```

**ESC/ESH Classification:**

| Classification | Systolic (mmHg) | Diastolic (mmHg) |
|---------------|-----------------|-------------------|
| Optimal | < 120 | and < 80 |
| Normal | 120–129 | or 80–84 |
| High Normal | 130–139 | or 85–89 |
| Hypertension Grade 1 | 140–159 | or 90–99 |
| Hypertension Grade 2 | 160–179 | or 100–109 |
| Hypertension Grade 3 | ≥ 180 | or ≥ 110 |

---

### Get Blood Pressure Aggregates

```
POST /partners/v1/health/blood_pressure/get
```

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `user_id` | string (UUID) | Yes | Partner's user identifier |
| `from` | string | Yes | Start date (`YYYY-MM-DD`) |
| `to` | string | Yes | End date (`YYYY-MM-DD`) |
| `granularity` | string | No | `daily` (default), `weekly`, `monthly`, `yearly` |

```bash
curl -X POST "https://ross.rolla.cloud/partners/v1/health/blood_pressure/get" \
  -H "Authorization: Bearer YOUR_ACCESS_TOKEN" \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "user_id=0af73d3e-e092-49f4-b81b-7cf50e9c7782&from=2024-11-01&to=2024-11-30&granularity=daily"
```

```json
{
  "success": true,
  "aggregates": [
    {
      "period_start": "2024-11-10",
      "avg_systolic_mm_hg": 118,
      "avg_diastolic_mm_hg": 78,
      "entry_count": 2
    }
  ],
  "min_entry": { "..." : "..." },
  "max_entry": { "..." : "..." }
}
```

---

### Get Blood Pressure History

```
POST /partners/v1/health/blood_pressure/history
```

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `user_id` | string (UUID) | Yes | Partner's user identifier |
| `from` | string | Yes | Start date (`YYYY-MM-DD`) |
| `to` | string | Yes | End date (`YYYY-MM-DD`) |
| `limit` | integer | No | Max entries (1–1000) |

```bash
curl -X POST "https://ross.rolla.cloud/partners/v1/health/blood_pressure/history" \
  -H "Authorization: Bearer YOUR_ACCESS_TOKEN" \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "user_id=0af73d3e-e092-49f4-b81b-7cf50e9c7782&from=2024-11-01&to=2024-11-30&limit=100"
```

```json
{
  "success": true,
  "history": [
    {
      "id": 42,
      "systolic_mm_hg": 120,
      "diastolic_mm_hg": 80,
      "timestamp_tz": "2024-11-10 08:00:00+00",
      "note": null,
      "source": "rolla"
    }
  ],
  "min_entry": { "..." : "..." },
  "max_entry": { "..." : "..." }
}
```

---

### Update Blood Pressure Entry

```
POST /partners/v1/health/blood_pressure/update
```

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `user_id` | string (UUID) | Yes | Partner's user identifier |
| `id` | integer | Yes | Blood pressure entry ID |
| `systolic` | integer | Yes | Systolic in mmHg (70–250) |
| `diastolic` | integer | Yes | Diastolic in mmHg (40–150) |
| `timestamp` | integer | Yes | Epoch milliseconds |
| `note` | string | No | Optional note (max 500 characters) |

```bash
curl -X POST "https://ross.rolla.cloud/partners/v1/health/blood_pressure/update" \
  -H "Authorization: Bearer YOUR_ACCESS_TOKEN" \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "user_id=0af73d3e-e092-49f4-b81b-7cf50e9c7782&id=42&systolic=118&diastolic=78&timestamp=1755043140000"
```

```json
{
  "success": true
}
```

---

### Delete Blood Pressure Entry

```
POST /partners/v1/health/blood_pressure/delete
```

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `user_id` | string (UUID) | Yes | Partner's user identifier |
| `id` | integer | Yes | Blood pressure entry ID |

```bash
curl -X POST "https://ross.rolla.cloud/partners/v1/health/blood_pressure/delete" \
  -H "Authorization: Bearer YOUR_ACCESS_TOKEN" \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "user_id=0af73d3e-e092-49f4-b81b-7cf50e9c7782&id=42"
```

```json
{
  "success": true
}
```

---

## Insights

### Get Insight History

```
POST /partners/v1/generated_insights/history
```

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `user_id` | string (UUID) | Yes | Partner's user identifier |
| `limit` | integer | No | Max results (1–100, default: 10) |

```bash
curl -X POST "https://ross.rolla.cloud/partners/v1/generated_insights/history" \
  -H "Authorization: Bearer YOUR_ACCESS_TOKEN" \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "user_id=0af73d3e-e092-49f4-b81b-7cf50e9c7782&limit=10"
```

```json
{
  "success": true,
  "history": [
    {
      "id": 123,
      "title": "Sleep Quality Insight",
      "content": "Your sleep quality has improved...",
      "created_at": "2024-11-10 08:00:00"
    }
  ]
}
```

---

### Get Insights from Date

```
POST /partners/v1/generated_insights/historyfrom
```

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `user_id` | string (UUID) | Yes | Partner's user identifier |
| `from` | string | Yes | Start date (`YYYY-MM-DD`) |

```bash
curl -X POST "https://ross.rolla.cloud/partners/v1/generated_insights/historyfrom" \
  -H "Authorization: Bearer YOUR_ACCESS_TOKEN" \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "user_id=0af73d3e-e092-49f4-b81b-7cf50e9c7782&from=2024-11-01"
```

```json
{
  "success": true,
  "history": [
    {
      "id": 123,
      "title": "Sleep Quality Insight",
      "content": "Your sleep quality has improved...",
      "created_at": "2024-11-10 08:00:00"
    }
  ]
}
```

---

### Rate Insight

```
POST /partners/v1/generated_insights/rate
```

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `user_id` | string (UUID) | Yes | Partner's user identifier |
| `id` | integer | Yes | Insight ID |
| `weight` | integer | Yes | `-1` (negative) or `1` (positive) |

```bash
curl -X POST "https://ross.rolla.cloud/partners/v1/generated_insights/rate" \
  -H "Authorization: Bearer YOUR_ACCESS_TOKEN" \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "user_id=0af73d3e-e092-49f4-b81b-7cf50e9c7782&id=123&weight=1"
```

```json
{
  "success": true
}
```

---

## Device Info

### Get Device Info

```
POST /partners/v1/device/info
```

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `user_id` | string (UUID) | Yes | Partner's user identifier |

```bash
curl -X POST "https://ross.rolla.cloud/partners/v1/device/info" \
  -H "Authorization: Bearer YOUR_ACCESS_TOKEN" \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "user_id=0af73d3e-e092-49f4-b81b-7cf50e9c7782"
```

```json
{
  "success": true,
  "device": {
    "id": 42,
    "mac_address": "AA:BB:CC:DD:EE:FF"
  }
}
```

---

### Get Band Sync Timestamps

```
POST /partners/v1/device/band_timestamps
```

Returns the last sync timestamps for each data type. Use this to determine what data still needs to be synced.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `user_id` | string (UUID) | Yes | Partner's user identifier |

```bash
curl -X POST "https://ross.rolla.cloud/partners/v1/device/band_timestamps" \
  -H "Authorization: Bearer YOUR_ACCESS_TOKEN" \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "user_id=0af73d3e-e092-49f4-b81b-7cf50e9c7782"
```

```json
{
  "success": true,
  "timestamps": {
    "steps_last_block": null,
    "steps_last_entry": null,
    "active_kcal_last_timestamp": null,
    "sleep_last_block": null,
    "sleep_last_entry": null,
    "hrv_last_timestamp": null,
    "activity_hr_last_block": null,
    "activity_hr_last_entry": null,
    "passive_hr_last_timestamp": null
  }
}
```

---

## Onboarding

### Get Onboarding Status

```
POST /partners/v1/onboarding/status
```

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `user_id` | string (UUID) | Yes | Partner's user identifier |

```bash
curl -X POST "https://ross.rolla.cloud/partners/v1/onboarding/status" \
  -H "Authorization: Bearer YOUR_ACCESS_TOKEN" \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "user_id=0af73d3e-e092-49f4-b81b-7cf50e9c7782"
```

```json
{
  "success": true,
  "status": {
    "isActive": true,
    "completed": false,
    "uniqueDaysCount": 3,
    "currentDayNumber": 3,
    "startedAt": "2025-10-08 07:20:11+00",
    "completedAt": null,
    "lastSyncedLocalDate": "2025-10-10"
  }
}
```

**Status fields:**

| Field | Description |
|-------|-------------|
| `isActive` | `true` if onboarding started and not completed (1–7 days) |
| `completed` | `true` when 8 unique days have been recorded |
| `uniqueDaysCount` | Integer 0–8 (monotonic counter) |
| `currentDayNumber` | Equals `uniqueDaysCount` when active; `0` when not started or completed |
| `startedAt` / `completedAt` | UTC timestamps |
| `lastSyncedLocalDate` | Most recent recorded date (`YYYY-MM-DD`) |

---

## Parameter Reference

### Common Parameters

| Parameter | Format | Description |
|-----------|--------|-------------|
| `user_id` | String (UUID) | Partner's user identifier, e.g. `0af73d3e-e092-49f4-b81b-7cf50e9c7782`. Required for all data endpoints. |
| `from` | `YYYY-MM-DD` | Start date (inclusive, user's local timezone) |
| `to` | `YYYY-MM-DD` | End date (inclusive, user's local timezone) |
| `type` | String | Aggregation granularity — varies by endpoint (see below) |
| `limit` | Integer | Results per page (activity list: 1–100; weight/BP history: 1–1000) |
| `cursor` | String | Opaque pagination token (activity list only) |
| `score` | String | Score type: `health`, `activity`, `readiness`, `steps`, `move_hours`, `active_points`, `sleep`, `rhr`, `hrv` |
| `granularity` | String | Weight/BP aggregation: `daily`, `weekly`, `monthly`, `yearly` |

### Type Parameter Values

| Endpoint | Valid `type` values |
|----------|-------------------|
| Heart Rate, HRV | `10_min`, `daily`, `monthly` |
| Steps, Calories, Move Hours, Active Points | `hourly`, `daily`, `monthly` |
| Sleep | `all` (individual sessions), `daily`, `monthly` |
| RHR, Score | `daily`, `monthly` |

---

## Response Field Units

| Field | Unit | Description |
|-------|------|-------------|
| `duration` | seconds | Length of an activity or time period |
| `distance` | meters | Distance covered during an activity |
| `calories` | kilocalories (kcal) | Calories burned |
| `steps` | count (integer) | Number of steps taken |
| `heart_rate` | beats per minute (bpm) | Heart rate measurement |
| `hrv` | milliseconds (ms) | Heart rate variability |
| `rhr` | beats per minute (bpm) | Resting heart rate |
| `move_hours` | count (integer) | Hours with > 250 steps recorded |
| `active_points` | points (integer) | Active points score for a given day |
| `systolic_mm_hg` / `diastolic_mm_hg` | mmHg | Blood pressure readings |
| `weight` | kilograms (kg) | Body weight |

---

**Prev:** [User Management](03-user-management.md) | **Next:** [Error Handling](05-error-handling.md) | **Home:** [README](README.md)
