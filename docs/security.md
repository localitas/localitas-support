# Security

## Data in Transit

All tunnel traffic is encrypted using TLS. Connections between your machine and our relay infrastructure use authenticated, encrypted channels.

## Data at Rest

We do not store your request/response data. The only data we store is:
- Account information (email, name, hashed password)
- Tunnel configuration (subdomain, port mappings)
- Billing records (managed by Stripe)

## Authentication

- Passwords are hashed with bcrypt (cost 14)
- Sessions use signed JWTs with 24-hour expiry
- API keys use signed JWTs with 1-year expiry and can be revoked at any time
- CSRF protection on all web forms

## Account Security

- Email verification on signup
- Password reset via email with 1-hour expiry tokens
- Account deletion with 30-day recovery window
- Rate limiting on authentication endpoints

## Core and Cloud Auth

Localitas uses a local-first auth model. Your local Core server manages its own users and passwords independently. The cloud (localitas.com) has its own separate account system.

- **Use the same email and password** when creating accounts on both your local Core and the cloud.
- **Password changes on Core sync to the cloud** automatically. The sync sends a bcrypt hash (never plaintext) and is authenticated with your SaaS API token.
- **The cloud never pushes credentials to Core.** Core always works offline without depending on cloud availability.
- **Password resets on the cloud only affect cloud login.** If you reset on the cloud, you must also reset on Core separately.

This design ensures your local system is never dependent on cloud availability for authentication.

## Infrastructure

- SaaS hosted on DigitalOcean
- PostgreSQL for data storage
- Stripe for payment processing (PCI compliant)
- No third-party analytics or tracking

## Reporting Vulnerabilities

If you find a security vulnerability, please email security@localitas.com. Do not open a public issue.
