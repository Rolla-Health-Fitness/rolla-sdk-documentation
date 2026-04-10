# Overview

The Rolla SDK authenticates users through a standard registration and login flow. Your app registers users, obtains access tokens via login, and passes those tokens to the SDK when initializing it.

---

## Base URLs

| Environment | Base URL | Use |
|-------------|----------|-----|
| Production | `https://ross.rolla.cloud` | Release builds, live data |
| Sandbox (RnD) | `https://ross-rnd.rolla.cloud` | Development and QA |

Use the sandbox environment during development. Switch to production for release builds.

---

## How It Works

1. **Obtain your Partner ID** — contact [support@rolla.app](mailto:support@rolla.app) during onboarding. You receive a `partner_id` string that identifies your organization.
2. **Register the user** — your app calls `POST /api/register` with the user's email and password to create their Rolla account.
3. **Log in** — your app calls `POST /api/login` with the user's email, password, and your `Partner-ID` header to obtain an access token and refresh token.
4. **Initialize the SDK** — pass the access token, refresh token, and expiry to `RollaConfiguration` and call `show()`.
5. **Handle token refresh** — when the token expires, the SDK notifies your app via a callback. Your app calls `POST /api/refresh_token` to get a fresh token and pushes it back to the SDK via `updateToken()`.

> For detailed request/response schemas for each endpoint, see [Authentication](02-authentication.md).

---

## Partner ID

The `partner_id` is a fixed identifier for your organization, provided by Rolla during onboarding. It is **not a secret** — it is safe to embed in your mobile app. You include it as the `Partner-ID` header on authentication requests and as the `partnerId` parameter in `RollaConfiguration`.

> **Note:** Do not confuse the `partner_id` with the Partner API credentials (`partner_secret`). The Partner API is a separate, optional server-to-server integration for data retrieval and user management — it is not required for SDK integration. Contact Rolla if you need server-to-server API access.

---

## Environments

| Value | Description |
|-------|-------------|
| `"production"` | Live / release builds |
| `"rnd"` | Development and QA (sandbox environment) |

If omitted in `RollaConfiguration`, defaults to `"rnd"`.

> **Why `"rnd"`?** The name stands for "Research and Development" — it is the SDK's internal label for the non-production sandbox environment.

When using the sandbox environment, all authentication requests should go to `https://ross-rnd.rolla.cloud`. When using production, use `https://ross.rolla.cloud`.

---

**Next:** [Authentication](02-authentication.md) | **Home:** [README](README.md)
