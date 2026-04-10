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
| `401` | Unauthorized | Missing, invalid, or expired token. Obtain a new token via `POST /partners/v1/token`. |
| `403` | Forbidden | Access denied to the requested resource. Verify the user belongs to your partner account. |
| `404` | Not Found | The endpoint or resource does not exist. Check the URL path. |
| `405` | Method Not Allowed | Wrong HTTP method used (e.g., GET instead of POST). Check the endpoint documentation. |
| `429` | Too Many Requests | Rate limit exceeded. Back off and retry after a delay. |
| `500` | Internal Server Error | Unexpected server-side error. Retry with exponential backoff. |

---

## Best Practices

1. **Retry with exponential backoff on `429` and `500`** — These are transient errors. Start with a 1-second delay and double it on each retry, up to a reasonable maximum (e.g., 60 seconds).

2. **Re-authenticate on `401`** — When you receive a `401 Unauthorized`, your token has expired. Obtain a new token via `POST /partners/v1/token` and retry the original request.

3. **Validate inputs before sending** — Check that required parameters are present and in the correct format (e.g., `user_id` is a valid UUID, dates are `YYYY-MM-DD`) before making API calls. This avoids unnecessary `400` errors.

4. **Use HTTPS exclusively** — Never send credentials or tokens over unencrypted connections.

5. **Log non-sensitive data only** — Log request/response metadata for debugging, but never log `partner_secret`, access tokens, or user credentials.

6. **Implement circuit breakers** — If the API returns repeated `500` errors, temporarily stop making requests to avoid overwhelming the server.

---

## Integration Checklist

Before going to production, verify the following:

- [ ] `partner_id` and `partner_secret` are stored securely (environment variables or secrets manager, not in source code)
- [ ] Token caching is implemented — you are not requesting a new token for every API call
- [ ] `401` errors trigger automatic re-authentication
- [ ] `400` errors are logged with the `reason` field for debugging
- [ ] `429` and `500` errors are retried with exponential backoff
- [ ] All API communication uses HTTPS
- [ ] No credentials or tokens are logged
- [ ] User data is validated before sending to the API
- [ ] The `partner_secret` is never exposed in client-side code

---

**Prev:** [Data Endpoints](04-data-endpoints.md) | **Home:** [README](README.md)
