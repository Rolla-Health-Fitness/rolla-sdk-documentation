# User Management

Manage partner users through registration, authentication, status checks, and disconnection. All endpoints require a valid Bearer token in the `Authorization` header.

---

> **Important:** The root README mentions that registration requires name, DOB, weight, height, gender, and timezone. This is **not accurate for the API**. The Partner API registration endpoint only requires `user_id` and `email`. Profile data (name, biometrics, timezone, etc.) is collected within the Rolla SDK UI after the user opens the app — not at registration time.

---

## Register User

```
POST /partners/v1/users/register
```

Registers a new user with your partner account. This creates the user in Rolla's system and associates them with your partner.

### Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `user_id` | string | Yes | The user's unique identifier in your system |
| `email` | string | Yes | The user's email address |

### Example Request

```bash
curl -X POST "https://ross.rolla.cloud/partners/v1/users/register" \
  -H "Authorization: Bearer YOUR_ACCESS_TOKEN" \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "user_id=users_id_in_your_system&email=user@example.com"
```

### Example Response

```json
{
  "success": true
}
```

---

## Authenticate & Connect User

```
POST /partners/v1/users/user_login
```

Authenticates an existing Rolla user and creates the partner-user mapping. Use this when a user already has a Rolla account and wants to connect it to your partner app.

### Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `user_id` | string | Yes | The user's unique identifier in your system |
| `email` | string | Yes | The user's Rolla email address |
| `password` | string | Yes | The user's Rolla account password |

### Example Request

```bash
curl -X POST "https://ross.rolla.cloud/partners/v1/users/user_login" \
  -H "Authorization: Bearer YOUR_ACCESS_TOKEN" \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "user_id=users_id_in_your_system&email=user@example.com&password=UsersPassword1"
```

### Example Response

```json
{
  "success": true,
  "message": "Account successfully connected"
}
```

---

## Check User Status

```
POST /partners/v1/users/check_status
```

Checks whether a user exists in Rolla's system and whether they are registered with your partner.

### Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `email` | string | Yes | The user's email address |

### Example Request

```bash
curl -X POST "https://ross.rolla.cloud/partners/v1/users/check_status" \
  -H "Authorization: Bearer YOUR_ACCESS_TOKEN" \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "email=user@example.com"
```

### Example Response (registered)

```json
{
  "success": true,
  "registered": true,
  "user_id": 123
}
```

### Example Response (exists but not registered with partner)

```json
{
  "success": false,
  "registered": false,
  "message": "User exists but has not been registered"
}
```

---

## Disconnect User

```
POST /partners/v1/users/disconnect
```

Disconnects a user from your partner account. The user's Rolla account remains active but is no longer associated with your partner.

### Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `email` | string | Yes | The user's email address |

### Example Request

```bash
curl -X POST "https://ross.rolla.cloud/partners/v1/users/disconnect" \
  -H "Authorization: Bearer YOUR_ACCESS_TOKEN" \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "email=user@example.com"
```

### Example Response

```json
{
  "success": true,
  "message": "Account disconnected"
}
```

---

**Prev:** [Authentication](02-authentication.md) | **Next:** [Data Endpoints](04-data-endpoints.md) | **Home:** [README](README.md)
