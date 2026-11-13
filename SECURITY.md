# Security

## Reporting a vulnerability

Please use GitHub's **private vulnerability reporting** on this repository.
Do not open a public issue for security problems.

## No secrets in this repo

This repo is public. It never contains:

- Tenant IDs, client IDs, object IDs or pool IDs
- Client secrets, certificates or tokens
- User names, mailboxes or screenshots with tenant data

Configuration comes from environment variables. Copy `.env.example` to `.env` and fill it locally.
`.env` is ignored by git.

Prefer certificate or federated credentials over client secrets.
