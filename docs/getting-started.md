# Getting Started

Localitas lets you run AI on your own hardware (Mac Minis, etc.) and access it securely from anywhere via tunnels.

## Prerequisites

- macOS (Apple Silicon)
- [Docker Desktop](https://www.docker.com/products/docker-desktop/) installed
- [Homebrew](https://brew.sh/) installed

## Step 1: Install

```bash
brew tap localitas/tap
brew install localitas-core localitas-worker
```

This installs the core engine and the MLX worker.

## Step 2: Start the core

```bash
brew services start localitas-core
```

Open [http://localhost:8090](http://localhost:8090) in your browser. Create your admin account on first visit.

## Step 3: Sign up for a Localitas account

Go to [localitas.com](https://localitas.com) and create an account. You'll receive a **90-day trial setup token** after registration.

## Step 4: Connect your machine

1. Copy the setup token shown after registration
2. In your local Localitas core, go to **Settings > Admin > SaaS Connection**
3. Paste the token and save

Your machine is now linked to the Localitas cloud.

## Step 5: Create a tunnel

Go to [Tunnels](https://localitas.com/tunnels) in the SaaS dashboard and create a tunnel. Your local machine gets a public HTTPS URL like `yourname.localitas.dev`.

## After the trial

Your trial token expires after 90 days. Subscribe to the [Family Plan ($20/mo)](https://localitas.com/billing) to keep tunnel access. The core engine and all 16 built-in apps remain free and self-hosted forever.

## Next Steps

- [How Tunnels Work](tunnels.md) - understand the networking
- [API Reference](api.md) - integrate with your apps
- [FAQ](faq.md) - common questions

## In-App Help

Once running, each Localitas app has built-in help accessible from within the app itself. The docs here cover product-level concepts; in-app help covers feature-specific usage.
