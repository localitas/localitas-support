# Vocalitas — Public Content Sharing

Vocalitas is the public-facing side of Localitas. While `*.localitas.com` is always behind login, `*.vocalitas.com` serves your content publicly — no account needed for visitors.

## Two doors to the same apps

Your family has one origin — the **family apex** — reachable through two doors:

- **`smith.localitas.com`** — the **private door** (login required). You, signed in, see everything: the dashboard and every app.
- **`smith.vocalitas.com`** — the **public door** (no login). The world sees only what you've explicitly published.

Same core, same apps, same data. The domain decides which door — and therefore what's visible. (`smith` is your family name; the owner user never appears in the URL.)

## Two flavors of "public"

**1. A public surface on an otherwise-private app** (a shared note, a public album). The app declares specific public paths; everything else 404s on the public door. No custom domain — it lives under the family origin.

| App | Public route | What visitors see |
|-----|-------------|-------------------|
| Notes | `smith.vocalitas.com/n/{uuid}` | Read-only published note |
| Albums | `smith.vocalitas.com/apps/albums/shared/{uuid}` | Public photo gallery (future) |

**2. A standalone public app** (a forum, a gallery, a social site — typically a vibe app declared `public`). It gets its **own subdomain** and its **own container**, and may bind your **own custom domain**.

| Shape | URL |
|-------|-----|
| Own subdomain | `forum.smith.vocalitas.com` |
| Your custom domain | `forum.yourbrand.com` (CNAME → `forum.smith.vocalitas.com`) |

## Publishing a note

1. Create a note in the Notes app
2. Call `POST /apps/notes/api/notes/{id}/publish`
3. You get back a `public_url` and `qr_url`
4. Share the vocalitas link — anyone can read it
5. Call `DELETE /apps/notes/api/notes/{id}/publish` to unpublish

## Custom domains (standalone public apps)

Point your own domain at a standalone public app so visitors see your brand, not ours:

1. Add the custom domain and note the CNAME target (`forum.smith.vocalitas.com`).
2. Create a CNAME record at your DNS provider pointing your domain at that target.
3. Verify — once DNS resolves, the domain is authorized and SSL is provisioned.

Custom domains apply only to standalone public apps (you can't point a domain at a shared-note *path*). Requires an active Family Plan subscription.

## Security

- Only routes explicitly registered as public are served on vocalitas — the app handler is the gatekeeper, not the domain.
- Spoofing the Host header doesn't help — unregistered routes return 404.
- Private routes (settings, admin, CRUD) are never accessible on vocalitas.
- A standalone public app runs in its own container on its own origin, isolated from your dashboard's session.
- A custom domain is authorized only after DNS verification proves you own it *and* it points at your own family's app.
