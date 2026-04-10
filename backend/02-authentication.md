# Authentication

Partners authenticate using **OAuth 2.0 Client Credentials**. You receive a unique `partner_id` and `partner_secret` during onboarding. These credentials are used to obtain a JWT access token, which must be included in the `Authorization` header of all subsequent API requests.

**Important:** You request a token once and cache it server-side. One token is used for all of your users — only the `user_id` parameter differs in data retrieval requests.

---

## Obtain Access Token

```
POST /partners/v1/token
```

> **⚠️ This endpoint only accepts `application/x-www-form-urlencoded` data. JSON request bodies are NOT supported.**

### Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `grant_type` | string | Yes | Must be `client_credentials` |
| `partner_id` | string | Yes | Your partner identifier |
| `partner_secret` | string | Yes | Your partner secret |

### Example Request

```bash
curl -X POST "https://ross.rolla.cloud/partners/v1/token" \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "grant_type=client_credentials&partner_id=your_id_here&partner_secret=your_secret_here"
```

### Example Response

```json
{
  "success": true,
  "access_token": "eyJ0eXAiOiJKV1Q...",
  "token_type": "Bearer",
  "expires_in": 3600,
  "partner_id": "your_partner_id"
}
```

---

## Validate Token

```
GET /partners/v1/token/validate
```

Validates an existing token. Use this to check whether your cached token is still valid before making data requests.

### Example Request

```bash
curl -X GET "https://ross.rolla.cloud/partners/v1/token/validate" \
  -H "Authorization: Bearer YOUR_ACCESS_TOKEN"
```

### Example Response

```json
{
  "success": true,
  "valid": true,
  "partner_id": "your_partner_id",
  "expires_at": "2025-11-10T08:25:28+00:00"
}
```

---

## Revoke Token

```
POST /partners/v1/token/revoke
```

Revokes a token before its natural expiry. Use this if a token may have been compromised or is no longer needed.

### Example Request

```bash
curl -X POST "https://ross.rolla.cloud/partners/v1/token/revoke" \
  -H "Authorization: Bearer YOUR_ACCESS_TOKEN"
```

### Example Response

```json
{
  "success": true
}
```

---

## Token Expiry

- **TTL:** 3600 seconds (1 hour)
- **No refresh endpoint.** Tokens cannot be refreshed before expiry. When your token expires, obtain a new one by calling `POST /partners/v1/token` again.

---

## Best Practices

1. **Cache the token server-side** — Store the token in memory or a secure cache. Reuse it for all user requests.
2. **Track expiry** — Use the `expires_in` value (or `expires_at` from `/token/validate`) to know when to request a new token. Request a new token shortly before expiry to avoid interruptions.
3. **Handle 401 automatically** — If any API request returns `401 Unauthorized`, your token has expired. Obtain a new token and retry the request.
4. **Never expose tokens to clients** — Access tokens are for server-to-server use only. Do not send them to mobile apps or web frontends.

---

**Prev:** [Overview](01-overview.md) | **Next:** [User Management](03-user-management.md) | **Home:** [README](README.md)
