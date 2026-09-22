# Project-Aware Discovery

The first useful response after a Funloom integration request should be based on local evidence, not a generic questionnaire.

## What To Inspect

Check the target frontend and backend separately when both exist:

- Framework and build system: Vite, React, Vue, Next.js, plain HTML, Workers, serverless, Express, Hono, etc.
- Package manager and scripts.
- `.env`, `.env.local`, deployment config, and whether client env vars are prefixed for the framework.
- Deployed env mechanism: Worker secrets, Pages variables, Vercel/Render variables, wrangler config, CI variables, and whether local env files are copied anywhere.
- Existing `@funloom/sdk`, `@funloom/cli`, lockfile entries, older local SDK path/Git dependencies, or CLI references.
- Existing Funloom platform developer-mode credentials or env names, such as `FUNLOOM_APP_ID`, `FUNLOOM_APP_SECRET`, and frontend public `VITE_FUNLOOM_APP_ID`.
- Existing auth: login/register routes, JWT/session/cookies, local-only guest id, user table, account settings UI.
- Account uniqueness and recovery: phone/email uniqueness, duplicate registration behavior, password reset or account restore, and binding conflict handling.
- Existing player identity: stable user id, generated guest id, nickname-only player, local storage profile.
- Existing wallet/item/currency system.
- Existing AI provider config, model selection, prompt pipeline, and server/client split.
- Existing payment/recharge/webhook code.
- UI surfaces where wallet, settings, inventory, start game, restart game, and AI actions live.
- Existing buttons/forms around recharge or start-game flows, especially whether buttons inside forms need `type="button"` or `preventDefault`.

## Question Pattern

Use this shape:

```text
我检查到 <具体文件/路由/状态>，目前 <项目事实>。
这会影响 Funloom 的 <绑定/扣钻/充值/AI/webhook>，因为 <原因>。
我建议 <具体方案>。
你确认这个方向吗？
```

Examples:

- "我检查到前端只有 `localStorage` 里的 guest id，后端没有登录接口。Funloom 绑定需要一个稳定 external user，所以我建议先补最小手机号/密码账号。你接受这个账号形态吗？"
- "我检查到项目已有 `wallet`/`items` 概念，所以我不会让 AI 直接扣钻。我建议新增一个可兑换道具作为游戏内消耗层。这个道具叫开局券、能量，还是沿用你现有道具名？"
- "我检查到已有 OpenAI provider 配置，所以我会保留它，再新增 Funloom provider 和手动切换开关。切换入口放设置页可以吗？"
- "我检查到没有后端或 serverless function。由于 `appSecret` 和扣钻不能进浏览器，我建议新增一个最小 serverless 层。你接受新增吗？"
- "我检查到 `package.json` 尚未安装 `@funloom/sdk`。根据 Funloom SDK 文档，当前公开接入使用 npm 包，所以我会用本项目的包管理器安装并更新锁文件。可以吗？"
- "我检查到项目里仍有本地路径/Git SDK 依赖。当前公开接入应使用 npm 包；除非你是在测未发布 fork，我建议切回 `@funloom/sdk`。可以吗？"
- "我检查到你已经从 Funloom 平台开发者模式拿到了 `appId` 和 `appSecret`。我会把 `appId` 放到前端公开变量，把 `appSecret` 只放到后端/serverless，本地先生成回调和充值返回路径；线上域名部署后再补真实地址。可以吗？"
- "我检查到本地有 `.env.local`，但部署配置在 Cloudflare/Vercel/Render 里是另一套。为了避免线上仍连旧环境，我会在交付说明里列出本地和线上分别要填的变量，并加一个环境诊断输出。"
- "我检查到注册接口允许重新创建看起来相同的账号，这会让 Funloom 绑定残留到旧用户。这里需要先确认账号唯一性和找回/换绑策略，而不是让用户删数据库。"

## Do Not Ask Blindly

Avoid these until after inspection:

- "你有没有后端？"
- "你有没有用户系统？"
- "你需要充值入口吗？"
- "你要不要显示余额？"
- "你要不要 webhook？"
- "SDK 应该怎么装？"
- "你现在连的是哪个端？"

Instead, inspect and state what exists, then ask about the next product decision or permission to add missing infrastructure.
