# Error Handling

All API errors return a JSON response with `success` set to `false` and a `reason` field describing the issue.

---

## Error Response Format

```json
{
  "success": false,
  "reason": "A description of what went wrong"
}
```

---

## HTTP Status Codes

| Status Code | Meaning | Description |
|-------------|---------|-------------|
| `400` | Bad Request | Invalid or missing parameters. Check the `reason` field for details. |
| `401` | Unauthorized | Missing, invalid, or expired token. Obtain a new token via `POST /api/login` or `POST /api/refresh_token`. Also returned if the `Partner-ID` header is missing or invalid. |
| `403` | Forbidden | Access denied to the requested resource. |
| `404` | Not Found | The endpoint or resource does not exist. Check the URL path. |
| `405` | Method Not Allowed | Wrong HTTP method used (e.g., GET instead of POST). Check the endpoint documentation. |
| `429` | Too Many Requests | Rate limit exceeded. Back off and retry after a delay. |
| `500` | Internal Server Error | Unexpected server-side error. Retry with exponential backoff. |

---

## Best Practices

1. **Retry with exponential backoff on `429` and `500`** — These are transient errors. Start with a 1-second delay and double it on each retry, up to a reasonable maximum (e.g., 60 seconds).

2. **Re-authenticate on `401`** — When you receive a `401 Unauthorized`, your token has expired. Call `POST /api/refresh_token` with your refresh token, or `POST /api/login` if the refresh token is also expired.

3. **Validate inputs before sending** — Check that required parameters are present and in the correct format before making API calls. This avoids unnecessary `400` errors.

4. **Use HTTPS exclusively** — Never send credentials or tokens over unencrypted connections.

5. **Log non-sensitive data only** — Log request/response metadata for debugging, but never log tokens, passwords, or user credentials.

---

## Integration Checklist

Before going to production, verify the following:

- [ ] `partner_id` is configured correctly for the production environment
- [ ] Token storage uses the platform's secure storage (iOS Keychain / Android Keystore)
- [ ] `401` errors trigger automatic token refresh
- [ ] `400` errors are logged with the `reason` field for debugging
- [ ] `429` and `500` errors are retried with exponential backoff
- [ ] All API communication uses HTTPS
- [ ] No tokens or passwords are logged
- [ ] `rollaDidRequestTokenRefresh` (iOS) / `onTokenExpired` (Android) is implemented
- [ ] `clearSession()` is called on user logout
- [ ] Production base URL (`https://ross.rolla.cloud`) is used for release builds

---

**Prev:** [Profile](03-profile.md) | **Home:** [README](README.md)
