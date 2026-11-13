# agent-365-hermes

> [!WARNING]
> **Experimental preview. Not production ready.**
> This project builds on preview services (Microsoft Agent 365, Windows 365 for Agents).
> APIs can change or break at any time. Use a test tenant only.

A [Hermes Agent](https://hermes-agent.nousresearch.com/) plugin that gives Hermes its own Microsoft Entra agent identity through [Microsoft Agent 365](https://learn.microsoft.com/microsoft-agent-365/developer/).
With that identity, Hermes can check out a [Windows 365 for Agents](https://learn.microsoft.com/windows-365/agents/introduction-windows-365-for-agents) Cloud PC, work on it, and release it.

## Status

Planning. See [PLAN.md](PLAN.md) for the phases and open questions.

## What you need

- A Microsoft 365 tenant in the Frontier program
- Windows 365 for Agents enabled, with a Cloud PC agent pool
- Hermes Agent installed locally

Setup instructions follow once the first phase works.

## Security

No secrets in this repo. See [SECURITY.md](SECURITY.md).

## License

[MIT](LICENSE)
