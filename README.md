# Keycloak Setup Guide

## Quick Start

### 1. Start Keycloak

```bash
cd keycloak
docker compose up -d
```

- **Keycloak Admin Console**: http://localhost:8080
- **Default admin**: `admin` / `admin`

### 2. Create Keycloak Client

See [Keycloak Client Setup](#keycloak-client-setup) below.


---

## Keycloak Client Setup

In realm `master`:

1. Go to **Clients** → **Create client**
2. **Client type**: `OpenID Connect`
3. **Client ID**: `nuxt-app`
4. Click **Next**
5. Configure capabilities:
   - **Client authentication**: `On` (confidential client)
   - **Authorization**: `Off`
   - **Standard flow**: `Off` (optional)
   - **Direct access grants**: `On` (required for password flow)
6. Click **Save**
7. Copy the generated **Client secret** from the **Credentials** tab

### Client URL Settings (optional)

- **Root URL**: `http://localhost:3000`
- **Home URL**: `http://localhost:3000/`
- **Web origins**: `http://localhost:3000`

### Logout Redirect

- **Valid post logout redirect URIs**: `http://localhost:3000/login`

---

## API Documentation

- All the API endpoint may refer here : https://www.keycloak.org/guides#getting-started
- Keycloack Postman API Collection : https://documenter.getpostman.com/view/7294517/SzmfZHnd
- Below are the API endpoint to use for login authentication

### POST Token (Login)

**Keycloak URL**

- Local: `http://localhost:8080/realms/master/protocol/openid-connect/token`
- Docker: `http://keycloak:8080/realms/master/protocol/openid-connect/token`

**Request**

| Header | Value |
|--------|-------|
| `Content-Type` | `application/x-www-form-urlencoded` |
| `Authorization` | `Basic {base64(client_id:client_secret)}` |

**Body** (form-urlencoded)

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `grant_type` | string | Yes | `password` (Direct Access Grant) |
| `username` | string | Yes | User email (used as username in Keycloak) |
| `password` | string | Yes | User password |
| `scope` | string | No | `openid profile email` |

**Success Response** `200 OK`

```json
{
  "access_token": "eyJhbGc...",
  "refresh_token": "eyJhbGc...",
  "id_token": "eyJhbGc...",
  "expires_in": 300,
  "token_type": "Bearer"
}
```

**Error Responses**

| Status | Description |
|--------|-------------|
| `400` | Invalid request (e.g. missing username/password) |
| `401` | Invalid credentials |

---

### GET UserInfo (Me)

**Keycloak URL**

- Local: `http://localhost:8080/realms/master/protocol/openid-connect/userinfo`
- Docker: `http://keycloak:8080/realms/master/protocol/openid-connect/userinfo`

**Request**

| Header | Value |
|--------|-------|
| `Authorization` | `Bearer {access_token}` |

**Success Response** `200 OK`

```json
{
  "sub": "abc-123-uuid",
  "email": "user@example.com",
  "name": "John Doe",
  "preferred_username": "johndoe"
}
```

| Field | Type | Description |
|-------|------|-------------|
| `sub` | string | Keycloak subject ID |
| `email` | string | Email address |
| `name` | string | Full name |
| `preferred_username` | string | Username |

---

### POST Logout

**Keycloak URL**

- Local: `http://localhost:8080/realms/master/protocol/openid-connect/logout`
- Docker: `http://keycloak:8080/realms/master/protocol/openid-connect/logout`

**Request**

| Header | Value |
|--------|-------|
| `Content-Type` | `application/x-www-form-urlencoded` |

**Body** (form-urlencoded)

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `refresh_token` | string | Yes | Refresh token to revoke |
| `client_id` | string | Yes | Client ID |
| `client_secret` | string | Yes | Client secret |

**Response**

- `204 No Content` on success.

**Browser redirect** (for end-session): Append `?post_logout_redirect_uri={url}&id_token_hint={id_token}` for redirect after logout.

---

## Environment Variables

### Keycloak (this folder)

| Variable | Default | Description |
|----------|---------|-------------|
| `KC_BOOTSTRAP_ADMIN_USERNAME` | `admin` | Admin console username |
| `KC_BOOTSTRAP_ADMIN_PASSWORD` | `admin` | Admin console password |

**Local development (no Docker)**

- `KEYCLOAK_INTERNAL_URL`: `http://localhost:8080`
- `KEYCLOAK_PUBLIC_URL`: `http://localhost:8080`

---

## Optional: LDAP Federation

To authenticate users from LDAP:

1. In Keycloak: **User federation** → **Add provider** → **ldap**
2. Configure: Connection URL, Bind DN, Bind credential, Users DN
3. Test connection and authentication
4. Save and sync/import users

The Nuxt app continues to use the same login flow; Keycloak delegates authentication to LDAP.

---

## Folder Layout

| Path | Description |
|------|-------------|
| `keycloak/docker-compose.yml` | Keycloak service only |
| `nuxt-app/docker-compose.yml` | Nuxt app (joins `keycloak_network`) |

---

## Stop Services

```bash
# Stop Keycloak
cd keycloak && docker compose down
```
