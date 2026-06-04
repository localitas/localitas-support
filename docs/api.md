# API Reference

The Localitas API lets you manage tunnels, domains, and billing programmatically.

## Authentication

All API requests require a Bearer token. Get your API key from the **API Keys** page in the dashboard.

```bash
curl -H "Authorization: Bearer YOUR_API_KEY" \
  https://app.localitas.com/api/tunnels
```

## Base URL

```
https://app.localitas.com/api
```

## Endpoints

### Tunnels

| Method | Path | Description |
|--------|------|-------------|
| GET | /tunnels | List your tunnels |
| POST | /tunnels | Create a tunnel |
| GET | /tunnels/{id} | Get tunnel details |
| DELETE | /tunnels/{id} | Delete a tunnel |
| GET | /tunnels/{id}/frp-config | Get FRP client config |

### Domains

| Method | Path | Description |
|--------|------|-------------|
| GET | /domains | List your domains |
| POST | /domains | Add a custom domain |
| POST | /domains/{id}/verify | Verify domain ownership |
| DELETE | /domains/{id} | Remove a domain |

### Billing

| Method | Path | Description |
|--------|------|-------------|
| GET | /billing/plans | List available plans |
| GET | /billing/subscription | Get current subscription |
| GET | /billing/history | Get invoice history |
| GET | /billing/payment-methods | List payment methods |
| POST | /billing/checkout | Start checkout (returns Stripe URL) |
| POST | /billing/portal | Open billing portal (returns Stripe URL) |

### Account

| Method | Path | Description |
|--------|------|-------------|
| GET | /auth/profile | Get your profile |
| PUT | /auth/profile | Update your profile |
| DELETE | /account | Delete your account (30-day recovery window) |
| POST | /account/recover | Recover a deleted account |

## Rate Limits

API requests are rate-limited per IP. Current limits are returned in response headers:

```
X-RateLimit-Limit: 60
X-RateLimit-Remaining: 58
X-RateLimit-Reset: 1700000000
```

## Errors

Errors return JSON with an `error` field:

```json
{
  "error": true,
  "message": "Invalid token",
  "timestamp": "2026-06-03T12:00:00Z"
}
```

## Interactive Docs

Full Swagger documentation is available at [app.localitas.com/api/docs](https://app.localitas.com/api/docs) when logged in.
