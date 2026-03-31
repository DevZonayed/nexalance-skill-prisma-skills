# CLI Auth — Reusing Prisma CLI Credentials

The Prisma CLI stores OAuth credentials after browser-based login. These tokens work with the Management API.

## Token file location

| Platform | Path |
|----------|------|
| macOS | `~/Library/Preferences/prisma-cli/auth.json` |
| Linux | `~/.config/prisma-cli/auth.json` |

## File format

```json
{
  "token": "<access_token_jwt>",
  "refreshToken": "<refresh_token_jwt>"
}
```

## Using the access token

The `token` field is a JWT with `workspace:admin` scope. Use it as a Bearer token:

```bash
curl -s -H "Authorization: Bearer $TOKEN" \
  https://api.prisma.io/v1/regions/postgres
```

If the API returns 401 (`authentication-failed`), the token has expired. Refresh it.

## Refreshing an expired token

Extract the `client_id` from the access token's JWT payload, then call the token endpoint:

```bash
curl -s -X POST https://auth.prisma.io/token \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "grant_type=refresh_token&client_id=<client_id>&refresh_token=<refresh_token>"
```

**Response:**

```json
{
  "access_token": "<new_access_token>",
  "refresh_token": "<new_refresh_token>",
  "expires_in": 3600,
  "scope": "workspace:admin offline_access",
  "token_type": "Bearer"
}
```

Use `access_token` for subsequent API calls. The refresh token is **rotated** — the old one becomes invalid after use.

## Extracting client_id from the JWT

The `client_id` is in the JWT payload (second segment, base64-decoded). In bash:

```bash
echo "$TOKEN" | cut -d. -f2 | base64 -d 2>/dev/null | python3 -c "import sys,json; print(json.load(sys.stdin)['client_id'])"
```

## When CLI credentials are not available

If the auth file doesn't exist, the user hasn't logged in via the Prisma CLI. Fall back to a service token (see `auth.md`).
