# CLAUDE.md

Guidance for Claude and other AI assistants working in this repo.

## Project

A Hermes Agent plugin that gives Hermes a Microsoft Entra agent identity through Agent 365.
Hermes uses it to check out Windows 365 for Agents Cloud PCs.
Experimental preview. Not production ready. See [PLAN.md](PLAN.md) for phases and status.

## Principles

- Start small. Build only what the current phase in PLAN.md needs.
- Follow official docs and best practices for Hermes and the Agent 365 SDK. Link to them, do not copy them.
- Flag anything that is preview only or poorly documented.

## Authorship

- Lukas is the author of every commit.
- Claude appears only as a `Co-Authored-By` trailer.

## Naming

Branches use a type prefix and a short kebab-case name:

| Prefix | Use |
|---|---|
| `feature/` | New functionality |
| `bugfix/` | Bug fixes |
| `refactor/` | Code changes without behavior change |
| `docs/` | Documentation only |
| `chore/` | Tooling, config, dependencies |

Example: `feature/token-provider`

Commit messages use a lowercase type prefix:

| Prefix | Use |
|---|---|
| `feat:` | New functionality |
| `bugfix:` | Bug fixes |
| `refactor:` | Code changes without behavior change |
| `docs:` | Documentation only |
| `chore:` | Tooling, config, dependencies |
| `test:` | Tests only |

Example: `feat: add Cloud PC checkout tool`

Pull request titles use a capitalized type prefix:

`Feature:`, `Bugfix:`, `Refactor:`, `Docs:`, `Chore:`, `Test:`

Example: `Feature: Add Cloud PC checkout tool`

## Writing style

Applies to docs, comments, commit messages and PR descriptions.

- Short sentences. One idea per sentence.
- Written for humans. No walls of text.
- No em dashes or en dashes.
- Every user facing doc keeps the preview disclaimer.

## Secrets

This repo is public. Never commit:

- Tenant IDs, client IDs, object IDs, pool IDs
- Secrets, certificates, tokens
- User names, mailboxes, or screenshots with tenant data

Use placeholders and `.env.example`. Real values live in a local `.env`. See [SECURITY.md](SECURITY.md).
