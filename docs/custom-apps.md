# Building Custom Apps

## Start with Vibe

Vibe is the built-in AI app builder. Open it from the Localitas launcher, pick a language (Go, Python, or Node.js), and describe what you want. Vibe generates a complete project with authentication, database, cron jobs, metrics, and deployment files.

**After Vibe scaffolds your project**, open it in VSCode and continue building with **Claude Code, Codex, or Gemini**. Vibe handles the platform boilerplate — your AI assistant handles the features.

### What Vibe generates

- Go/Python/Node.js HTTP server with routes
- SQLite database with migrations
- HTML/CSS UI using HTMX (no JavaScript frameworks)
- mDNS auto-discovery for the Localitas cluster
- Auth middleware that reads the platform bearer token
- Cron job declarations for scheduled tasks
- DogStatsD metrics reporting
- Docker deployment files
- Tests

## Security: Deploy in Docker

For maximum safety, deploy custom apps inside Docker containers. This ensures your app:

- **Cannot read** `~/.localitas/config-core.yaml` or other host files (API tokens, secrets)
- **Cannot access** other apps' SQLite databases directly
- **Communicates** with Core only via the HTTP API
- **Is discovered** automatically via Docker label scanning

```bash
make start-docker   # build image + run container
```

The scaffold's `Dockerfile` is already configured correctly. **Never mount `config-core.yaml` or `~/.localitas/` into the container.**

## Authentication

The Localitas platform proxy handles authentication. When a logged-in user accesses your app, the proxy injects an `Authorization` header:

```
Authorization: Bearer base64({"user_id":"...","email":"...","name":"...","permission":"..."})
```

The bearer token is a base64-encoded JSON payload with these fields:

| Field | Type | Description |
|-------|------|-------------|
| `user_id` | string | UUID of the authenticated user |
| `email` | string | User's email address |
| `name` | string | User's display name |
| `permission` | string | Highest permission: `admin`, `write`, `read`, or `guest` |

Your app does not need to manage users, sessions, or passwords. The platform handles all of that. You just read the token.

### Reading the token in your app

The Vibe scaffold includes auth middleware that parses this token automatically. In your handlers:

**Go:**
```go
userID := client.GetUserID(r.Context())     // "2b9af8b9-856a-..."
email := client.GetEmail(r.Context())       // "user@example.com"
permission := client.GetScope(r.Context())  // "admin"
```

**Python:**
```python
user = verify_token(request)
user_id = user["user_id"]
email = user["email"]
```

**Node.js:**
```javascript
const userId = req.userId;
const email = req.userEmail;
```

### Enforcing permissions

Restrict routes by permission level using scope middleware:

```go
mux.HandleFunc("GET /api/items", handleList)                                       // read (default)
mux.HandleFunc("POST /api/items", client.RequireScopeFunc(client.ScopeWrite, handleCreate))
mux.HandleFunc("DELETE /api/admin", client.RequireScopeFunc(client.ScopeAdmin, handleReset))
```

Scope hierarchy: `admin > write > read > guest`.

## Making a Vibe app public

Vibe apps can declare public routes accessible on `vocalitas.com` without login:

```go
func (a *App) PublicPaths() []string {
    return []string{"/apps/ext/myapp/public/*"}
}
```

## Deploying

1. App runs on your machine (Mac Mini, etc.)
2. Core discovers it via mDNS (native) or Docker labels (container)
3. Create a tunnel in the SaaS dashboard for remote access
4. Private URL: `myapp.smith.localitas.com`
5. Public URL: `myapp.smith.vocalitas.com` (for declared public routes)
