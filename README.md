# Funloom Game Integration Skill

This repository provides assistant skills for integrating web and H5 games with Funloom. It routes Funloom-hosted Advanced Custom Games to Hosted SDK 2.0 and externally deployed games to the Browser/Server SDK and OpenAPI flows.

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
- External-game onboarding should start from Funloom platform developer mode, where the creator obtains the external game `appId` and one-time `appSecret`. Hosted Advanced Custom Games do not need those external-game credentials.
- The assistant should inspect the target game project before asking detailed implementation questions.
