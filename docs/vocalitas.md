# Vocalitas — Public Content Sharing

Vocalitas is the public-facing side of Localitas. While `*.localitas.com` is always behind login, `*.vocalitas.com` serves your content publicly — no account needed for visitors.

## How it works

Every tunnel you create gets two URLs:

- **`myapp.smith.localitas.com`** — private, requires login (your admin view)
- **`myapp.smith.vocalitas.com`** — public, no login (what the world sees)

Same tunnel, same backend, same machine. The core app decides what to show based on which domain the request comes from.

## What can be public?

Each app declares which routes are accessible publicly. By default, everything is private. Apps opt-in to public routes.

| App | Public route | What visitors see |
|-----|-------------|-------------------|
| Notes | `/apps/notes/pub/{uuid}` | Read-only published note |
| Albums | `/apps/albums/shared/{uuid}` | Public photo gallery (future) |
| Vibe apps | `/apps/ext/{app}/public/*` | Whatever the builder decides |

## Publishing a note

1. Create a note in the Notes app
2. Call `POST /apps/notes/api/notes/{id}/publish`
3. You get back a `public_url` and `qr_url`
4. Share the vocalitas link — anyone can read it
5. Call `DELETE /apps/notes/api/notes/{id}/publish` to unpublish

## Security

- Only routes explicitly registered as public are served on vocalitas
- Spoofing the Host header doesn't help — unregistered routes return 404
- Private routes (settings, admin, CRUD) are never accessible on vocalitas
- The app handler is the gatekeeper, not the domain

## Custom domains (coming soon)

You can point your own domain (e.g., `myblog.com`) to your tunnel. Your visitors see your brand, not ours. Requires an active Family Plan subscription.
