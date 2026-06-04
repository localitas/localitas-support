# Getting Started

Localitas lets you run AI on your own hardware (Mac Minis, etc.) and access it securely from anywhere via tunnels.

## Prerequisites

- macOS (Apple Silicon recommended)
- Homebrew installed

## Install

```bash
brew tap localitas/tap
brew install localitas
```

## First Run

```bash
localitas start
```

This starts the core engine on your machine. Open [http://localhost:8080](http://localhost:8080) to access the local dashboard.

## Connect to the Cloud

To access your setup remotely, create an account at [localitas.com](https://localitas.com) and link your machine:

1. Sign up and subscribe to the Tunnel plan
2. Create an API key in the dashboard
3. Run:

```bash
localitas connect --api-key YOUR_API_KEY
```

Your local machine is now accessible via a secure tunnel with a custom subdomain.

## Next Steps

- [How Tunnels Work](tunnels.md) - understand the networking
- [API Reference](api.md) - integrate with your apps
- [FAQ](faq.md) - common questions

## In-App Help

Once running, each Localitas app has built-in help accessible from within the app itself. The docs here cover product-level concepts; in-app help covers feature-specific usage.
