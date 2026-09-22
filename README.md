# Funloom Game Integration Skill

This repository provides assistant skills for integrating existing web games with the Funloom SDK, OpenAPI, diamond wallet, recharge flow, item exchange, official AI, and webhook flows.

## Codex

The repository root is the Codex skill.

Install with:

```powershell
git clone https://github.com/EndlessNeo/funloom-game-integration.git "$env:USERPROFILE\.codex\skills\funloom-game-integration"
```

Use in Codex:

```text
$funloom-game-integration
```

## Claude Code

The Claude Code version is in:

```text
claude/funloom-game-integration
```

Install by copying that folder into your Claude skills directory:

```text
~/.claude/skills/funloom-game-integration
```

Then ask Claude Code to use `funloom-game-integration` for the integration task.

## Notes

- Keep app secrets, webhook secrets, payment keys, and model provider keys out of browser code and Git history.
- Public creator onboarding should start from Funloom platform developer mode, where the creator obtains the external game `appId` and one-time `appSecret`.
- The assistant should inspect the target game project before asking detailed implementation questions.
