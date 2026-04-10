# Overview

The Rolla Partner API enables third-party partners to securely access and synchronize user data from Rolla's platform. It uses the **OAuth 2.0 Client Credentials flow** for backend-to-backend authentication.

---

## Base URL

```
https://ross.rolla.cloud
```

All API endpoints are prefixed with `/partners/v1/`.

---

## Key Features

- Secure OAuth 2.0 Client Credentials authentication
- JWT-based access tokens
- Access to partner-owned user data and analytics
- Health, activity, and biometric data retrieval

---

## How Credentials Are Provisioned

Your `partner_id` and `partner_secret` are created as part of the onboarding process:

- **partner_id** — Manually chosen based on the client/partner name
- **partner_secret** — A randomly generated string, provided securely during onboarding

The provisioning process is scripted and initiated manually by the Rolla team. Credentials are delivered via a secure communication channel agreed upon during onboarding.

To request credentials, contact [support@rolla.app](mailto:support@rolla.app).

---

## Security Warning

> **⚠️ IMPORTANT: The `partner_secret` is for server-to-server communication ONLY.**
>
> **Never** include the `partner_secret` in client applications (mobile apps, web frontends, or any code that runs on end-user devices). Token requests must be made exclusively from your backend server.

---

## How It Works

1. Your backend authenticates with Rolla using `partner_id` and `partner_secret` to obtain a JWT access token
2. You cache the token server-side and reuse it for all API requests (one token covers all your users)
3. You pass the `user_id` parameter to retrieve data for individual users
4. When the token expires (after 1 hour), your backend requests a new one

---

**Next:** [Authentication](02-authentication.md) | **Home:** [README](README.md)
