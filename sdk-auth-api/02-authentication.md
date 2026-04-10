# Authentication

This section covers user registration, login, token refresh, and the complete token lifecycle for SDK integration.

All authentication endpoints use `application/x-www-form-urlencoded` request bodies and require the `Partner-ID` header.

---

## Register a User

```
POST /api/register
```

Creates a new user account in Rolla's system. Call this once per user — typically during your app's sign-up flow.

### Headers

| Header | Required | Description |
|--------|----------|-------------|
| `Partner-ID` | Yes | Your partner identifier |
| `Content-Type` | Yes | `application/x-www-form-urlencoded` |

### Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `email` | string | Yes | The user's email address |
| `password` | string | Yes | The user's chosen password |

### Example Request

```bash
curl -X POST "https://ross-rnd.rolla.cloud/api/register" \
  -H "Partner-ID: your-partner-id" \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "email=user@example.com&password=SecurePassword123"
```

### Example Response

```json
{
  "success": true
}
```

> **Note:** Profile data (name, date of birth, weight, height, gender, timezone) is **not** required at registration. The SDK collects this information through its built-in onboarding flow when the user first opens it.

---

## Log In

```
POST /api/login
```

Authenticates a registered user and returns an access token and refresh token. Use these tokens to initialize the SDK.

### Headers

| Header | Required | Description |
|--------|----------|-------------|
| `Partner-ID` | Yes | Your partner identifier |
| `Content-Type` | Yes | `application/x-www-form-urlencoded` |

### Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `email` | string | Yes | The user's email address |
| `password` | string | Yes | The user's password |

### Example Request

```bash
curl -X POST "https://ross-rnd.rolla.cloud/api/login" \
  -H "Partner-ID: your-partner-id" \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "email=user@example.com&password=SecurePassword123"
```

### Example Response

```json
{
  "success": true,
  "tokens": {
    "access_token": "eyJ0eXAiOiJKV1Q...",
    "token_type": "Bearer",
    "expires_in": 1800,
    "refresh_token": "abc123...",
    "refresh_expires_in": 2592000
  }
}
```

### Response Fields

| Field | Type | Description |
|-------|------|-------------|
| `access_token` | string | JWT access token — pass this to `RollaConfiguration.token` |
| `token_type` | string | Always `"Bearer"` |
| `expires_in` | int | Token lifetime in seconds (1800 = 30 minutes) |
| `refresh_token` | string | Token used to obtain a fresh access token — pass this to `RollaConfiguration.refreshToken` |
| `refresh_expires_in` | int | Refresh token lifetime in seconds (2592000 = 30 days) |

---

## Refresh Token

```
POST /api/refresh_token
```

Obtains a fresh access token using a valid refresh token. Call this when the SDK notifies your app that the token has expired (via `rollaDidRequestTokenRefresh` on iOS or `onTokenExpired` on Android).

> **Note:** The SDK attempts to refresh the token automatically using the refresh token you provided at initialization. This endpoint is for cases where the SDK's internal refresh fails, or when you need to refresh tokens outside of the SDK (e.g., during a background session refresh).

### Headers

| Header | Required | Description |
|--------|----------|-------------|
| `Partner-ID` | Yes | Your partner identifier |
| `Content-Type` | Yes | `application/x-www-form-urlencoded` |

### Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `refresh_token` | string | Yes | The refresh token from a previous login or refresh response |

### Example Request

```bash
curl -X POST "https://ross-rnd.rolla.cloud/api/refresh_token" \
  -H "Partner-ID: your-partner-id" \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "refresh_token=abc123..."
```

### Example Response

```json
{
  "success": true,
  "tokens": {
    "access_token": "eyJ0eXAiOiJKV1Q...",
    "token_type": "Bearer",
    "expires_in": 1800,
    "refresh_token": "def456...",
    "refresh_expires_in": 2592000
  }
}
```

> **Important:** Each refresh response returns a **new** refresh token. Always store and use the latest refresh token for subsequent refresh calls.

---

## Token Lifecycle Summary

```
1. Register user     POST /api/register        (once per user)
2. Log in            POST /api/login            → access_token + refresh_token
3. Initialize SDK    RollaConfiguration(token: access_token, refreshToken: refresh_token, ...)
4. SDK runs          (SDK uses access_token for internal API calls)
5. Token expires     SDK fires callback → your app calls POST /api/refresh_token
6. Update SDK        rolla.updateToken(token: new_access_token, refreshToken: new_refresh_token, ...)
7. Repeat 4-6
```

### Token Lifetimes

| Token | Lifetime | Notes |
|-------|----------|-------|
| Access token | 1800 seconds (30 minutes) | Used for all SDK API calls |
| Refresh token | 2592000 seconds (30 days) | Used to obtain fresh access tokens |

### Best Practices

1. **Always pass both tokens** — provide `refreshToken` and `tokenExpiresIn` in `RollaConfiguration` so the SDK can manage refresh automatically.
2. **Store tokens securely** — use the platform's secure storage (iOS Keychain, Android Keystore/EncryptedSharedPreferences).
3. **Handle the refresh callback** — always implement `rollaDidRequestTokenRefresh` (iOS) or `onTokenExpired` (Android) as a fallback.
4. **Use HTTPS exclusively** — never send credentials or tokens over unencrypted connections.
5. **Clear on logout** — call `rolla.clearSession()` when the user logs out of your app.

---

**Prev:** [Overview](01-overview.md) | **Next:** [Error Handling](03-error-handling.md) | **Home:** [README](README.md)
