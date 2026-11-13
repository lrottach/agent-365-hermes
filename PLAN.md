# Plan: Hermes connector for Windows 365 for Agents

> **Preview project.** Built on preview services. Not production ready.

## Goal

Give Hermes its own Microsoft Entra agent identity through Agent 365.
Let Hermes check out a Windows 365 Cloud PC, work on it, and release it.
Grow that into an app testing scenario.

## What the docs tell us

| Piece | What it is | Status |
|---|---|---|
| Hermes plugins | Python package in `~/.hermes/plugins/<name>/` or pip entry point `hermes_agent.plugins`. `plugin.yaml` manifest plus `register(ctx)`. Tools via `ctx.register_tool`, hooks via `ctx.register_hook`, env vars via `requires_env`. | Stable |
| Hermes MCP client | Native. `mcp_servers` in `~/.hermes/config.yaml`. Static headers or OAuth 2.1. No token refresh for static headers. | Stable |
| Agent identity | Entra Agent ID. Blueprint holds credentials. Agent identity is the runtime principal. Modes: S2S, OBO, Agentic User. | Agentic User needs Frontier |
| Agent 365 SDK (Python) | `microsoft-agents-a365-runtime`, `-tooling`, `-notifications`. Tooling extensions exist for OpenAI, SK, Agent Framework, ADK, Foundry. **None for Hermes.** | Preview |
| Agent 365 CLI and skills | `a365` CLI plus `microsoft/agent365-skills` (`make-a365-agent`) to register blueprint and identity. | Preview |
| W365 for Agents MCP | Server `mcp_W365ComputerUse` behind the Agent 365 gateway. Session tools `StartSession`, `EndSession`, `GetSessionDetails`, plus ~60 action tools (screenshot, click, type, browser, UI tree, shell). | Preview |
| W365 Session API | Direct REST alternative: `POST /api/pools/{poolId}/sessions`, `DELETE /api/sessions/{id}`. | Preview |

Key facts for the design:

- Sessions idle out after 30 minutes. Cloud PC resets on release. No state survives.
- Agent tokens expire independently of the session. They must be refreshed.
- Shell commands are allow listed and capped at 30 seconds. Browser DOM tools work only in Edge.
- No file read or write tool on the Cloud PC.

## Architecture

```
Hermes (model + loop)
  └─ hermes-w365 plugin (Python)
       ├─ identity     Agent 365 identity, token cache and refresh
       ├─ session      checkout, heartbeat, release, auto release on exit
       └─ tools        curated wrappers around mcp_W365ComputerUse
            │  Streamable HTTP MCP, agent token
            ▼
  Agent 365 tools gateway  ──►  Windows 365 for Agents pool  ──►  Cloud PC
```

Why a plugin and not just `mcp_servers` config: Hermes can only send static headers. Agent tokens expire. The plugin owns token refresh, the session lifecycle and the safety net (always release). It also lets us expose a small, curated tool set instead of 65 raw tools.

## Phases

### Phase 0: Prove the path (no plugin code)

- Register a blueprint and agent identity with `make-a365-agent`.
- Create a W365 provisioning policy (agents) and a small pool.
- Add `mcp_W365ComputerUse` with `a365 develop add-mcp-servers`. Admin grants consent.
- Get a token manually. Point Hermes `mcp_servers` at the gateway with a static header.
- Ask Hermes to start a session, take a screenshot, end the session.

**Done when:** one manual checkout, screenshot and release works end to end. Auth mode (S2S vs Agentic User) is confirmed.

### Phase 1: Minimal plugin (identity + one checkout)

- Plugin skeleton: `plugin.yaml`, `register(ctx)`, `requires_env` for tenant ID, blueprint client ID, agent ID.
- Token provider with cache and refresh. Certificate or federated credential. No client secrets in examples.
- Tools: `w365_checkout`, `w365_release`, `w365_status`, `w365_screenshot`.
- Hook: release the session when the Hermes session ends.
- Heartbeat while a session is open.
- Unit tests with mocked MCP responses.

**Done when:** `hermes plugins doctor` passes. Hermes checks out a Cloud PC, returns a screenshot, releases it. A crash still releases the session.

### Phase 2: Let Hermes work on the Cloud PC

- Wrap a curated action set: click, type, keys, list and activate windows, launch app, UI tree, find UI element, OCR.
- A passthrough tool for anything not wrapped, gated by config.
- A Hermes skill that teaches the loop: observe, act, verify, release.

**Done when:** Hermes opens an app from the Start menu on its own and reports what it saw.

### Phase 3: App testing scenario

- Skill: "verify app deployment" with a short test spec (YAML) as input.
- Target app: a self built SAP style demo app with mock data. No real database or network backend.
- Checks: Start menu entry, desktop shortcut, launch, sign in, mock data visible, installed version.
- "Database connection" means the app's own status indicator, not a real DB check.
- Output: a Markdown report with screenshots, pass or fail per check.

**Done when:** one run against the sample app produces a usable report.

### Later (only if needed)

- Agentic User mode and Teams or Outlook entry point (Frontier).
- Screenshare link for human take control.
- Observability with Microsoft OpenTelemetry.

## Repo structure

```
agent-365-hermes/
├── README.md              short pitch, preview banner, quick start
├── PLAN.md                this file
├── SECURITY.md            how to report issues, no secrets policy
├── .env.example           placeholder settings only
├── plugin/                the Hermes plugin (pip installable)
│   ├── pyproject.toml
│   └── hermes_w365/
│       ├── plugin.yaml
│       ├── __init__.py    register(ctx)
│       ├── identity.py
│       ├── session.py
│       └── tools.py
├── skills/                Hermes skills (Phase 2 and 3)
├── examples/              sample config, sample test spec
├── docs/
│   ├── setup.md           tenant setup, step by step
│   ├── architecture.md    one diagram, one page
│   └── blog-notes.md      running notes for the blog post
└── tests/
```

## Contribution

- Lukas is the author of every commit. Claude appears only as a `Co-Authored-By` trailer.

## Docs approach

- Preview banner at the top of README, docs and the plugin description.
- One topic per page. Short sentences. Diagrams over prose.
- `setup.md` grows with each phase. Written while building, not after.
- Link to official docs instead of copying them.

## Secret hygiene (public repo)

- No tenant IDs, client IDs, UPNs or pool IDs in code, docs or screenshots. Use placeholders.
- Config from env vars or `~/.hermes/.env`. Ship `.env.example` only.
- `.gitignore` for `.env`, certs, token caches, test reports, screenshots.
- Enable GitHub secret scanning and push protection. Add a gitleaks pre-commit hook and CI step.
- Prefer certificate or federated credentials. Document client secrets as dev only, never committed.
- Redact tokens in logs. Never log headers.
- Screenshots in reports may show tenant data. Keep reports out of the repo.

## Feeding the blog

- `docs/blog-notes.md` gets a short entry per phase: what worked, what surprised, one screenshot idea.
- Capture a screen recording at the end of Phase 1 and Phase 3.
- Blog storyline: "an AI agent with its own identity borrows a Cloud PC to test an app release."

## Open questions and risks

1. **Auth mode.** W365 docs say agents use an agent user identity. That implies Agentic User mode, which needs Frontier. The target tenant is in Frontier and W365 for Agents is enabled, so this path is open. Phase 0 still confirms whether S2S also works, since it is simpler.
2. **Token flow in Python.** Agent 365 docs show tooling with OpenAI and SK extensions, not a raw token call. The exact Python path for a custom framework is thin. Fallback is MSAL with the blueprint credential.
3. **Gateway vs Session API.** Two documented paths to checkout. Plan uses the gateway (fits "Agent 365 SDK"). Session API is the fallback.
4. **Pool selection.** `StartSession` parameters for choosing a pool are not documented in detail.
5. **Licensing and cost.** W365 for Agents is pay as you go. Keep pools small and always release sessions.
6. **Hermes plugin API** is young and changing. Pin a Hermes version in docs.
7. **Hosting.** Hermes runs locally. The standard agent path needs no public endpoint. Agentic User and notifications would.

## Sources

- Hermes plugins: https://hermes-agent.nousresearch.com/docs/developer-guide/plugins
- Hermes MCP: https://hermes-agent.nousresearch.com/docs/user-guide/features/mcp
- Agent 365 SDK: https://learn.microsoft.com/microsoft-agent-365/developer/agent-365-sdk
- Agent 365 identity: https://learn.microsoft.com/microsoft-agent-365/developer/identity
- Agent 365 dev lifecycle: https://learn.microsoft.com/microsoft-agent-365/developer/a365-dev-lifecycle
- Agent 365 tooling: https://learn.microsoft.com/microsoft-agent-365/developer/tooling
- W365 for Agents MCP reference: https://learn.microsoft.com/microsoft-copilot-studio/mcp-windows-365-agents
- W365 for Agents API overview: https://learn.microsoft.com/windows-365/agents/mcp-tool-overview
- W365 session lifecycle: https://learn.microsoft.com/windows-365/agents/agent-session-lifecycle
- W365 identity and security: https://learn.microsoft.com/windows-365/agents/identity-security
