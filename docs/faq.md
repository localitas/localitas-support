# FAQ

## General

**What is Localitas?**
Localitas is a platform for running AI on your own hardware and accessing it securely from anywhere. Instead of paying for cloud compute, you run models on machines you own (like Mac Minis) and connect to them via secure tunnels.

**What hardware do I need?**
Any Mac with Apple Silicon (M1 or later). Mac Mini is the best value for a dedicated setup.

**Is my data private?**
Yes. Your AI models and data stay on your hardware. We only relay encrypted traffic through our tunnels. We never store or inspect your request/response data.

## Tunnels

**What happens if my internet goes down?**
Your tunnel disconnects. It will automatically reconnect when your internet comes back.

**Can I use my own domain?**
Yes. Add a custom domain in the Domains section of the dashboard and verify ownership via DNS.

**What's the bandwidth limit?**
100 MB/s per tunnel. This is the relay throughput; your actual speed depends on your home internet upload bandwidth.

## Billing

**Can I cancel anytime?**
Yes. Your subscription stays active until the end of the billing period. No refunds for partial periods.

**What happens if I delete my account?**
Your account is deactivated immediately. You have 30 days to recover it. After 30 days, all data is permanently deleted.

**Do you offer a free tier?**
Not currently. We offer a single plan at $20/month or $200/year.

## Troubleshooting

**My tunnel says "disconnected"**
1. Check that your local service is running
2. Check that the FRP client is running on your machine
3. Check your internet connection

**I'm getting 502 errors**
Your local service isn't responding on the configured port. Make sure it's running and listening on the correct port.

**I didn't get the verification email**
Check your spam folder. If it's not there, try registering again or contact support.

## Vocalitas (Public Sharing)

**What is vocalitas.com?**
Vocalitas is the public-facing domain for your Localitas content. While `*.localitas.com` requires login, `*.vocalitas.com` serves your shared content publicly — no account needed for visitors.

**How do I share a note publicly?**
In the Notes app, publish a note via the API. You'll get a public URL on vocalitas.com and a QR code. Anyone with the link can read it.

**Can visitors access my private data through vocalitas?**
No. Only routes that apps explicitly declare as public are accessible on vocalitas. Everything else returns 404. Private routes like settings, admin, and edit pages are never served on vocalitas.

**Can I use my own domain?**
Custom domain support is coming soon. You'll be able to point `myblog.com` to your tunnel with automatic SSL via Let's Encrypt.

## Custom Apps

**Can I build my own app?**
Yes. Use Vibe to generate an app from a plain English description, or build a standalone Go app from scratch. See [How to Create a Custom App](custom-apps.md).

**Does my custom app need to be in Go?**
For Vibe-generated apps, yes. For standalone apps, you can use any language — just register via mDNS and serve HTTP. The Go libraries for public paths and SQLite replication are optional but recommended.

**Can my custom app be public on vocalitas?**
Yes. Declare public paths using the `publicpath` library, and those routes will be accessible on vocalitas.com without login.
