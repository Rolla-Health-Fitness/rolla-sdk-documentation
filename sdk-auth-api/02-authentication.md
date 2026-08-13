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

> **Note:** Profile data (name, date of birth, weight, height, gender, timezone) is **not** required at registration. The SDK collects this information through its built-in onboarding flow when the user first opens it — or your app can set it in advance via [Profile](03-profile.md) so the onboarding is skipped.

---

## Log In

```
POST /api/login
```

Authenticates a registered user and returns an access token and refresh token. Pass all three values — `access_token`, `refresh_token`, and `expires_in` — into `RollaConfiguration` when initializing the SDK.

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
| `expires_in` | int | Access token lifetime in seconds (1800 = 30 minutes) — pass this to `RollaConfiguration.tokenExpiresIn` |
| `refresh_token` | string | Token used to obtain a fresh token pair — pass this to `RollaConfiguration.refreshToken` |
| `refresh_expires_in` | int | Refresh token lifetime in seconds (2592000 = 30 days) |

---

## Refresh Token

```
POST /api/refresh_token
```

Exchanges the **latest** refresh token for a fresh token pair — a new access token *and* a new refresh token. Refresh tokens are single-use, so this call only ever succeeds with the newest token in the chain and permanently invalidates the token you sent in the request.

> **Who calls this endpoint:** in normal operation, only the SDK. It refreshes automatically with the refresh token it currently holds — initially the one from `RollaConfiguration`, afterwards the ones from its own rotations — and delivers every new pair to your app via `rollaDidRefreshToken` (iOS) / `onTokenRefreshed` (Android). Call this endpoint yourself only if your app owns the token rotation (for example your backend keeps the user's Rolla session alive server-side) — and in that case push every pair you obtain to the SDK with `updateToken()` immediately, because the pair the SDK still holds is invalidated the moment your call succeeds.

> **When the SDK reports an expired session** — `rollaDidRequestTokenRefresh` (iOS) / `onTokenExpired` (Android) — its own refresh has just **failed**: the refresh token it held is consumed or expired. Unless your app holds a newer, unused refresh token, this endpoint fails for you too. Recover by re-authenticating with [`/api/login`](#log-in) and pushing the new pair to the SDK via `updateToken()`.

### Headers

| Header | Required | Description |
|--------|----------|-------------|
| `Partner-ID` | Yes | Your partner identifier |
| `Content-Type` | Yes | `application/x-www-form-urlencoded` |

### Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `refresh_token` | string | Yes | The **latest** refresh token — from the most recent login or refresh response. Older tokens in the chain are already invalid |

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

> **Important:** Refresh tokens are **single-use**. Each refresh response returns a **new** refresh token, and the one just used is permanently invalidated — reusing it fails with HTTP 401. Always store and use the latest refresh token, and always pass the latest pair to the SDK (see Token Management: [Android](../android/06-token-management.md) / [iOS](../ios/07-token-management.md)).

---

## Token Lifecycle Summary

```
1. Register user      POST /api/register   (once per user)
2. Log in             POST /api/login   →  access_token + refresh_token + expires_in
3. Initialize SDK     RollaConfiguration(token, refreshToken, tokenExpiresIn)
                      — always built from the newest pair your app has stored
4. SDK runs           The SDK refreshes its own tokens as needed and delivers every
                      new pair via onTokenRefreshed / rollaDidRefreshToken
                      → your app persists it and uses it for every later step 3
5. SDK cannot refresh onTokenExpired / rollaDidRequestTokenRefresh fires
                      → your app re-authenticates (POST /api/login) and calls
                        rolla.updateToken(token, refreshToken, expiresIn)
6. Log out            rolla.clearSession()
```

### Token Lifetimes

| Token | Lifetime | Notes |
|-------|----------|-------|
| Access token | 1800 seconds (30 minutes) | Used for all SDK API calls |
| Refresh token | 2592000 seconds (30 days) | Used to obtain fresh access tokens |

### Best Practices

1. **Pass all three token fields** — `token`, `refreshToken`, and `tokenExpiresIn` in `RollaConfiguration`, always from the newest pair your app has stored. Without `refreshToken` the SDK cannot refresh at all; without `tokenExpiresIn` it can only recover after the first 401.
2. **Persist every rotated pair** — implement `rollaDidRefreshToken` (iOS) / `onTokenRefreshed` (Android), store the delivered pair, and build every later `RollaConfiguration` from it. The pair it replaces is invalid from that moment.
3. **Answer the expired callback** — implement `rollaDidRequestTokenRefresh` (iOS) / `onTokenExpired` (Android): re-authenticate and push the new pair with `updateToken()`. This is the only in-place recovery once the SDK's own refresh fails.
4. **Store tokens securely** — use the platform's secure storage (iOS Keychain, Android Keystore/EncryptedSharedPreferences).
5. **Use HTTPS exclusively** — never send credentials or tokens over unencrypted connections.
6. **Clear on logout** — call `rolla.clearSession()` when the user logs out of your app.

The platform Token Management guides ([Android](../android/06-token-management.md) / [iOS](../ios/07-token-management.md)) cover the full integration contract, with code examples for every callback.

---

**Previous:** [Overview](01-overview.md) | **Next:** [Profile](03-profile.md) | **Home:** [README](README.md)
