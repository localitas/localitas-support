# How to Create a Custom App

Localitas supports two ways to build custom apps: using Vibe (AI-powered) or building from scratch.

## Using Vibe

Vibe is the built-in AI app builder. Describe what you want in plain English, and Vibe generates a complete Go application with database, API, and UI.

### Steps

1. Open the **Vibe** app from the sidebar
2. Describe your app: "Build a booking system with calendar, customer list, and email reminders"
3. Vibe generates the code, database schema, and UI
4. Review and deploy — your app gets its own route under `/apps/ext/{name}/`
5. Create a tunnel to make it accessible from the internet

### What Vibe generates

- Go HTTP server with routes
- SQLite database with migrations
- HTML/CSS UI using HTMX (no JavaScript frameworks)
- mDNS auto-discovery for the Localitas cluster
- Public path declarations for vocalitas sharing

### Making a Vibe app public

Vibe apps can declare public routes that are accessible on `vocalitas.com` without login:

```go
func (a *App) PublicPaths() []string {
    return []string{"/apps/ext/myapp/public/*"}
}
```

Routes under `/public/` are served on vocalitas. Everything else requires login via localitas.

## Building a Custom App from Scratch

For full control, build a standalone Go application that integrates with the Localitas platform.

### Requirements

1. **Go HTTP server** — your app runs as its own process on its own port
2. **mDNS registration** — so the core discovers your app on the local network
3. **SQLite for data** — use the Raft-replicated SQLite client for data that syncs across nodes
4. **Public path declarations** — if you want vocalitas support

### Minimal standalone app

```go
package main

import (
    "net/http"
    "github.com/localitas/localitas/go/internal/publicpath"
)

func main() {
    registry := publicpath.NewRegistry()
    registry.Add("/public/*")

    mux := http.NewServeMux()
    mux.HandleFunc("/", handleHome)
    mux.HandleFunc("/public/{id}", handlePublicView)

    http.ListenAndServe(":9201", mux)
}
```

### mDNS registration

Your app must register with the core via mDNS so it appears in the app sidebar and can be discovered by other nodes:

```go
import "github.com/localitas/localitas/go/internal/discovery"

discovery.Register("myapp", 9201, map[string]string{
    "version": "1.0",
    "type":    "standalone",
})
```

### Public paths for vocalitas

Use the `publicpath` package to declare which routes are public:

```go
import "github.com/localitas/localitas/go/internal/publicpath"

registry := publicpath.NewRegistry()
registry.Add(
    "/public/*",           // all public routes
    "/shared/{uuid}",      // specific shared items
)
```

In your handlers, check the Host header to decide rendering:

```go
func handlePublicView(w http.ResponseWriter, r *http.Request) {
    host := r.Host
    if strings.Contains(host, "vocalitas") {
        // Render clean public template
        renderPublicView(w, r)
        return
    }
    // Render admin view with edit controls
    renderAdminView(w, r)
}
```

### Database

Use the Raft-replicated SQLite client for data that should sync across cluster nodes:

```go
import "github.com/localitas/localitas/go/internal/sqlitedb"

client, err := sqlitedb.NewRaftClient(sqlitedb.RaftClientConfig{
    AppName: "myapp",
    DataDir: dataDir,
})
```

### Tunnel setup

After your app is running:

1. Go to the SaaS dashboard → **Tunnels**
2. Create a tunnel pointing to your app's port
3. Your app is live at `myapp.smith.localitas.com`
4. Public routes are live at `myapp.smith.vocalitas.com`

### App structure

```
myapp/
├── cmd/myapp/main.go          # Entrypoint
├── internal/
│   ├── handlers.go            # HTTP handlers
│   ├── storage.go             # SQLite queries
│   └── templates/             # HTML templates
├── migrations/
│   └── 001_init.sql           # Database schema
├── docs/
│   └── help.md                # In-app help (shown in Localitas UI)
├── go.mod
└── Makefile
```

### Help documentation

Add a `docs/help.md` file with frontmatter to show help inside the Localitas UI:

```markdown
---
title: My App
---

# My App

Description of what your app does and how to use it.
```

## Deploying

Both Vibe and custom apps deploy the same way:

1. App runs on your machine (Mac Mini)
2. Core discovers it via mDNS
3. Create a tunnel in the SaaS dashboard
4. Public URL: `myapp.smith.localitas.com` (private) / `myapp.smith.vocalitas.com` (public)
5. Custom domain: point `myblog.com` CNAME to your tunnel (coming soon)
