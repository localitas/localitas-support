# How Tunnels Work

Localitas tunnels give you secure remote access to services running on your local machine.

## Overview

When you create a tunnel, Localitas establishes a persistent connection between your machine and our relay infrastructure. Incoming requests to your subdomain (e.g., `my-ai.localitas.com`) are forwarded to your local service.

```
Internet → localitas.com → encrypted tunnel → your machine:8080
```

## What You Get

- **Custom subdomain** - `yourname.localitas.com`
- **HTTPS included** - TLS termination handled automatically
- **100 MB/s bandwidth** per tunnel
- **Unlimited tunnels** on the Tunnel plan

## How It Works

Localitas uses [FRP](https://github.com/fatedier/frp) (Fast Reverse Proxy) under the hood:

1. Your machine runs an FRP client that connects to our relay server
2. The relay server receives incoming HTTP/HTTPS requests for your subdomain
3. Requests are forwarded through the encrypted tunnel to your local port
4. Responses travel back the same path

All traffic is encrypted in transit. We never store your request/response data.

## Creating a Tunnel

From the dashboard:

1. Go to **Tunnels** in the sidebar
2. Click **Create Tunnel**
3. Choose a subdomain and local port
4. Copy the FRP client config to your machine

Or via the CLI:

```bash
localitas tunnel create --subdomain my-ai --port 8080
```

## Custom Domains

You can also point your own domain at a tunnel. See the **Domains** section in the dashboard.

## Troubleshooting

- **Tunnel shows "disconnected"** - check that your local service is running and the FRP client is connected
- **502 errors** - your local service isn't responding on the configured port
- **Slow responses** - check your upload bandwidth; tunnels are limited by your home internet speed
