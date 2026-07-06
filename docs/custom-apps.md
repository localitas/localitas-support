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

Store your GitHub PAT in the Vault app, then reference its ID in your config. The vault credential must have these keys in its data:

| Key | Value | Example |
|-----|-------|---------|
| `username` | GitHub username | `octocat` |
| `password` | GitHub PAT with `read:packages` scope | `ghp_xxxxxxxxxxxx` |

Create the credential via the Vault API:

```bash
curl -X POST http://localhost:8080/api/apps/vault/api/credentials \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "GitHub Container Registry",
    "url": "https://ghcr.io",
    "data": {
      "username": "your-github-username",
      "password": "ghp_xxxxxxxxxxxxxxxxxxxx"
    }
  }'
```

The response includes a `public_id`. Add it to your config:

```yaml
core:
  container_registry:
    vault_credential_id: "the-public-id-from-response"
```

At pull time, Localitas fetches the decrypted `username` and `password` from the vault credential's secrets endpoint.

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

## Vault Integration for App Configuration

Third-party apps often need secrets (API keys, database URLs, tokens). Instead of hardcoding them in compose YAML, store them in the Vault app and let Localitas inject them at container start.

### How it works

1. Store your app's configuration as key-value pairs in a Vault credential
2. Reference the credential's `public_id` in your compose YAML
3. At container start, Localitas fetches the secrets and injects them two ways:
   - **Environment variables**: all key-values, with dots converted to underscores and uppercased
   - **Config files**: all key-values expanded into hierarchical YAML and bind-mounted into the container

### Step 1: Create a vault credential

Store your app's config as flat dot-separated key-value pairs:

```bash
curl -X POST http://localhost:8080/api/apps/vault/api/credentials \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "My Weather App Config",
    "data": {
      "database.host": "localhost",
      "database.port": "5432",
      "database.name": "weather",
      "api_key": "sk-xxxxxxxxxxxx",
      "cache.ttl": "300",
      "cors.origins.0": "https://example.com",
      "cors.origins.1": "https://api.example.com"
    }
  }'
```

Note the `public_id` in the response — you'll need it in the compose YAML.

### Step 2: Reference in compose YAML

```yaml
services:
  app:
    image: ghcr.io/myorg/weather-app:latest
    vault_credential_id: "the-public-id"
    config_mounts:
      - container_path: "/app/config.yaml"
```

### What the container receives

**Environment variables** (automatic from all vault keys):

```
DATABASE_HOST=localhost
DATABASE_PORT=5432
DATABASE_NAME=weather
API_KEY=sk-xxxxxxxxxxxx
CACHE_TTL=300
CORS_ORIGINS_0=https://example.com
CORS_ORIGINS_1=https://api.example.com
```

**Config file at `/app/config.yaml`** (dot-paths expanded to hierarchical YAML):

```yaml
database:
  host: localhost
  port: 5432
  name: weather
api_key: sk-xxxxxxxxxxxx
cache:
  ttl: 300
cors:
  origins:
    - https://example.com
    - https://api.example.com
```

### Multiple config files

If your app needs separate config files for different components, use a different vault credential for each mount:

```yaml
services:
  app:
    image: ghcr.io/myorg/myapp:latest
    vault_credential_id: "cred_main"
    config_mounts:
      - container_path: "/app/config.yaml"
      - vault_credential_id: "cred_database"
        container_path: "/app/database.yaml"
```

- `/app/config.yaml` uses all secrets from `cred_main`
- `/app/database.yaml` uses all secrets from `cred_database`
- Environment variables come from `cred_main` (the service-level credential)

### Key format rules

| Vault key | Env var | YAML output |
|-----------|---------|-------------|
| `api_key` | `API_KEY` | `api_key: value` |
| `database.host` | `DATABASE_HOST` | `database:\n  host: value` |
| `items.0` | `ITEMS_0` | `items:\n  - value` |
| `users.0.name` | `USERS_0_NAME` | `users:\n  - name: value` |
| `"true"` | (string) | boolean `true` in YAML |
| `"3000"` | (string) | integer `3000` in YAML |
| `'[1,2,3]'` | (string) | list `[1, 2, 3]` in YAML |

### Security

- Vault credentials are encrypted at rest (AES-256)
- Secrets are replicated across cluster nodes via Raft (encrypted in transit)
- Config files are written to temp directory with 0600 permissions
- Config files are bind-mounted read-only into the container
- Environment variables are only visible inside the container

## Remote LLM Providers

The AI chat can use external LLM CLI tools (Claude, Antigravity, Codex) for DAG builder inference instead of local MLX models. This is useful for baseline testing — if a cloud model can't pick the right tool with your prompt, the prompt is broken. Antigravity (Gemini 3.5 Flash) is free.

### Configuration

Enable providers in `config-core.yaml` (disabled by default):

```yaml
core:
  remote_llm:
    - name: claude
      enabled: false
      path: /opt/homebrew/bin/claude       # Claude Code CLI
    - name: antigravity
      enabled: false
      path: /opt/homebrew/bin/agy          # Antigravity CLI (Gemini)
    - name: codex
      enabled: false
      path: /opt/homebrew/bin/codex        # OpenAI Codex CLI
```

Then set the DAG builder model to use a provider:

```yaml
core:
  llm:
    models:
      - id: "antigravity:gemini-3.5-flash"  # free tier
        role: dag_builder
        max_tokens: 1024
```

Available model IDs: `antigravity:gemini-3.5-flash` (free), `antigravity:gemini-2.5-pro`, `claude:sonnet`, `claude:haiku`, `claude:opus`, `codex:o3`, `codex:o4-mini`.

### How it works

1. Each node advertises its available CLI providers in Raft cluster state
2. When a chat request comes in, the DAG builder sends `POST /api/ai/remote-llm` to a node that has the provider
3. That node shells out to the CLI binary and returns the response
4. If no node has the provider, the request falls back to the local node

### Adding a new provider

No code changes needed. Add it to the config, set the model ID with the `provider:model` prefix. Unknown providers get a `-p prompt` CLI fallback. If your CLI needs special arguments, add a case in `buildCLIArgs()`.

## AI Chat Tool Results — Presentation API

When a user asks the AI assistant a question, the DAG builder routes it to your app's API endpoint. Your endpoint can return rich, interactive HTML that renders directly in the chat message.

### How it works

1. The DAG executor calls your endpoint with `Caller: llm` header
2. Your endpoint detects this header and returns `Content-Type: text/markdown`
3. The response body is markdown containing embedded HTML, CSS, and JS
4. The chat UI renders it inside a **Shadow DOM** container — fully isolated
5. Your `<script>` block is executed in the global scope (`window`)
6. Your `<style>` block is scoped to your widget — no conflicts

### Architecture

```
Tool endpoint                    Chat UI
  │                                │
  │  Content-Type: text/markdown   │
  │  Body: <style>...<div>...      │
  │        <script>...             │
  ├───────────────────────────────►│
  │                                │
  │                    ┌───────────┴──────────┐
  │                    │  Shadow DOM host     │
  │                    │  ┌────────────────┐  │
  │                    │  │ <style> scoped │  │
  │                    │  │ <div> content  │  │
  │                    │  └────────────────┘  │
  │                    │  <script> → window   │
  │                    └──────────────────────┘
```

**CSS** lives inside the Shadow DOM — fully isolated from the chat UI and other widgets. **JavaScript** is executed in the global scope (`window`) so `onclick` attributes can find your functions.

### Template

Your widget has three parts: `<style>`, HTML, and `<script>`. Use Go's `html/template` and `go:embed` to build it — never `fmt.Sprintf` for HTML.

```html
<style>
.results { display:flex; flex-direction:column; gap:0.25rem; }
.row { display:flex; align-items:center; gap:0.5rem; padding:0.375rem 0.5rem;
       border-radius:0.375rem; cursor:pointer; }
.row:hover { background:var(--color-glass-border); }
.badge { color:var(--color-text-secondary); font-size:0.75rem; }
</style>

<div class="results">
  <div class="row" onclick="myappOpen('item-1')">
    <span class="badge">Type</span>
    <span>Item Name</span>
  </div>
</div>

<script>
function myappOpen(id) {
    // Overlay appended to document.body (escapes Shadow DOM for fullscreen)
    var overlay = document.createElement('div');
    overlay.style.cssText = 'position:fixed;inset:0;z-index:9999;background:rgba(0,0,0,0.85);display:flex;align-items:center;justify-content:center;';

    var frame = document.createElement('iframe');
    frame.src = '/apps/myapp/api/detail?id=' + id;
    frame.style.cssText = 'width:90vw;height:90vh;border:none;border-radius:0.75rem;';
    overlay.appendChild(frame);
    document.body.appendChild(overlay);

    // Close on click outside or ESC
    overlay.onclick = function(e) { if (e.target === overlay) close(); };
    function close() { overlay.remove(); document.removeEventListener('keydown', k); }
    function k(e) { if (e.key === 'Escape') close(); }
    document.addEventListener('keydown', k);
}
</script>
```

### Implementation in Go

Use `go:embed` for the template file, `html/template` for rendering:

```go
//go:embed templates/partials/_my_widget.html
var myWidgetFS embed.FS

var myWidgetTmpl = template.Must(
    template.New("widget").Parse(mustReadTemplate()),
)

func (a *App) handleEndpoint(w http.ResponseWriter, r *http.Request) {
    data := fetchData(r)

    if strings.EqualFold(r.Header.Get("Caller"), "llm") {
        w.Header().Set("Content-Type", "text/markdown")
        var buf bytes.Buffer
        myWidgetTmpl.Execute(&buf, data)
        w.Write(buf.Bytes())
        return
    }

    writeJSON(w, r, http.StatusOK, data)
}
```

### Key rules

| Rule | Why |
|------|-----|
| `<style>` is scoped to Shadow DOM | Your CSS can't conflict with the chat UI or other widgets |
| `<script>` runs in global scope (`window`) | `onclick` attributes resolve from `window`, not Shadow DOM |
| Prefix JS function names with your app abbreviation (`myappOpen`, `fsPreview`) | Multiple widgets in the same chat share `window` — avoid collisions |
| Overlays/modals append to `document.body` | They need to escape the Shadow DOM boundary for fullscreen |
| Use `this.getRootNode()` in event handlers to query Shadow DOM elements | `document.getElementById` won't find elements inside Shadow DOM |
| Use `html/template` + `go:embed` for widget templates | Never `fmt.Sprintf` for HTML — causes escaping bugs with `%` characters |
| Use CSS variables from the theme | Platform injects them into each Shadow DOM root |

### Available CSS variables

```
--color-text-primary       Main text color
--color-text-secondary     Muted text color
--color-bg-base            Background color
--color-glass-base         Glass panel background
--color-glass-border       Border/hover color
--color-success            Green (confirmation)
--color-error              Red (errors)
```

### Querying elements inside Shadow DOM

In `onclick` handlers, `document.querySelector` won't find elements inside your Shadow DOM. Use `this.getRootNode()`:

```js
// Pagination button inside Shadow DOM
function myappPage(btn, direction) {
    var root = btn.getRootNode();          // gets the Shadow DOM root
    var list = root.getElementById('list'); // finds element inside shadow
    // ... paginate ...
}
```

### Fullscreen overlays

Overlays need to escape the Shadow DOM to cover the full page. Append to `document.body`:

```js
function myappPreview(id) {
    var overlay = document.createElement('div');
    overlay.style.cssText = 'position:fixed;inset:0;z-index:9999;...';
    document.body.appendChild(overlay);  // outside Shadow DOM

    // Close handler
    function close() {
        overlay.remove();
        document.removeEventListener('keydown', k);
    }
    overlay.onclick = function(e) { if (e.target === overlay) close(); };
    function k(e) { if (e.key === 'Escape') close(); }
    document.addEventListener('keydown', k);
}
```

### Common patterns

**Navigation arrows + keyboard + swipe:**
```js
function k(e) {
    if (e.key === 'Escape') close();
    else if (e.key === 'ArrowRight') nav(1);
    else if (e.key === 'ArrowLeft') nav(-1);
}
document.addEventListener('keydown', k);

// Touch swipe
var tx = 0;
overlay.ontouchstart = function(e) { tx = e.touches[0].clientX; };
overlay.ontouchend = function(e) {
    var dx = e.changedTouches[0].clientX - tx;
    if (Math.abs(dx) > 50) { dx > 0 ? nav(-1) : nav(1); }
};
```

**Client-side filtering:**
```js
// Filter input inside Shadow DOM
function myappFilter(input) {
    var rows = input.getRootNode().querySelectorAll('.row');
    var q = input.value.toLowerCase();
    rows.forEach(function(r) {
        r.style.display = r.dataset.name.toLowerCase().indexOf(q) >= 0 ? 'flex' : 'none';
    });
}
```

### What NOT to do

- Don't load external scripts (CDN, analytics) — everything must be inline
- Don't use `document.write` — the DOM is already loaded
- Don't define JS functions without an app prefix — `window` is shared across widgets
- Don't use `document.getElementById` to find Shadow DOM elements — use `this.getRootNode()`
- Don't use `fmt.Sprintf` to build HTML in Go — use `html/template`
- Don't forget to clean up `keydown` listeners when closing overlays
