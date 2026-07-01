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

## App Store Deployment

The App Store is the recommended way to deploy apps. It manages Docker containers across your cluster using `docker-compose.yml` files.

### Every app includes

- **`docker-compose.yml`** — defines the service, ports, memory limits, and labels
- **`make docker-push`** — runs tests, builds, and pushes the image to `ghcr.io/localitas/localitas-app-{name}`

### Install via CLI

```bash
localitas-core app-store add --name myapp --compose ./docker-compose.yml --port 9000
localitas-core app-store start myapp
```

### Install via UI

Open the App Store (package icon, top-right nav, admin only). Paste the `docker-compose.yml`, set the port, and click "Add App".

### How it works

1. Core stores the compose YAML in Raft-replicated SQLite
2. On start, Core parses the YAML and creates containers via the Docker API
3. Containers are labeled `localitas.app=true` for automatic discovery
4. The app appears in the launcher within seconds
5. Each app gets one port, used identically across all cluster nodes
6. Same port on multiple nodes = automatic load balancing

### Remote access

1. Create a tunnel in the SaaS dashboard
2. Private URL: `myapp.smith.localitas.com`
3. Public URL: `myapp.smith.vocalitas.com` (for declared public routes)

## Container Registry Authentication

When Localitas starts an app from the App Store, it pulls the Docker image from the container registry. Public images work without authentication. Private images (e.g., private GitHub packages) require credentials.

### Authentication methods (in priority order)

**1. Config file credentials**

Set GitHub username and a Personal Access Token (PAT) with `read:packages` scope in your config:

```yaml
core:
  container_registry:
    github_username: "your-github-username"
    github_token: "ghp_xxxxxxxxxxxxxxxxxxxx"
```

Or use the CLI:

```bash
localitas-core config set core.container_registry.github_username "your-username"
localitas-core config set core.container_registry.github_token "ghp_xxxxxxxxxxxxxxxxxxxx"
```

**2. Vault app credentials**

Store your GitHub PAT in the Vault app with the tag `ghcr`. Localitas will fetch it automatically at pull time:

```yaml
core:
  container_registry:
    vault_credential_id: "your-vault-credential-id"
```

Or just create a credential in the Vault app with the tag `ghcr` — Localitas searches by tag if no explicit credential ID is set.

**3. Docker CLI login (fallback)**

If neither config nor vault credentials are set, Localitas uses whatever Docker credentials are available locally. Log in via the Docker CLI:

```bash
docker login ghcr.io -u YOUR_GITHUB_USERNAME -p ghp_xxxxxxxxxxxxxxxxxxxx
```

Note: On macOS with Docker Desktop, credentials are stored in the system keychain via the `osxkeychain` credential helper. The Go Docker SDK cannot read keychain credentials directly — use method 1 or 2 instead.

**4. No authentication (public images)**

Public images on `ghcr.io` or Docker Hub pull without any credentials.

### Creating a GitHub PAT for container registry

1. Go to GitHub Settings > Developer Settings > Personal Access Tokens > Fine-grained tokens
2. Create a token with the `read:packages` permission
3. Set the resource owner to the organization that owns the packages
4. Copy the token and add it to your Localitas config or Vault app
