# Deployment And Environment

Use this when a game behaves differently locally and online, when the creator cannot tell which Funloom endpoint is active, or when deployment tooling warns that local config will override remote config.

## Env Surfaces To Separate

Do not collapse these into one "env":

- local frontend env, such as Vite `.env.local` and `VITE_` variables
- local backend env
- deployed frontend build variables
- deployed Worker/serverless runtime secrets
- platform dashboard config
- generated sample files such as `.env.example`

Local `.env.local` files do not update deployed environments. CLI `--write-env` is useful for local development, but hosted environments still need their own secrets or build variables.

## Current Environment Fingerprint

When diagnosing, report a safe fingerprint:

- Funloom API base
- Funloom frontend base
- app id prefix or suffix, not the secret
- origin
- redirect URI
- webhook URL
- deployment target, such as local dev, preview, staging, or production
- package source for the SDK

Do not print app secrets, JWT secrets, payment keys, private certs, or full tokens.

## Funloom Endpoint Pairing

Keep these meanings separate:

- `FUNLOOM_API_BASE_URL`: Funloom OpenAPI/backend host, for server SDK calls such as auth code exchange, wallet, recharge session, diamond consumption, and AI.
- `FUNLOOM_FRONTEND_BASE_URL`: Funloom platform frontend host, for browser pages such as external authorization and external recharge.
- External game URL/origin: the game's own deployed site, used for redirect/callback/return URLs and allowed origins.

Do not fill `FUNLOOM_FRONTEND_BASE_URL` with the external game's URL. Do not infer the Funloom platform frontend from a Cloudflare Worker name or from the current browser URL.

For standard Funloom OpenAPI hosts, the platform backend may infer the matching frontend for a server-created recharge `checkoutUrl`:

- test OpenAPI host -> test Funloom platform frontend
- production OpenAPI host -> production Funloom platform frontend

This inference does not create the Browser SDK authorization URL. When the Browser SDK calls `buildAuthorizeUrl()` or `buildCheckoutUrl()`, pass the correct `FUNLOOM_FRONTEND_BASE_URL` (or legacy `authBaseUrl`) explicitly; its `apiBaseUrl` fallback is only compatibility behavior. Do not turn environment selection into an onboarding question for ordinary creators. Use explicit server frontend/recharge overrides only for non-standard preview/custom environments or while repairing a missing platform mapping.

## Framework Env Rules

Inspect the framework before explaining env behavior:

- Vite exposes only variables prefixed with `VITE_` to browser code.
- Next.js exposes only `NEXT_PUBLIC_` variables to browser code.
- Workers and serverless functions read runtime bindings/secrets, not frontend `.env.local`.
- Static frontend deployments often bake env values at build time.

If the deployed app still points to an old Funloom endpoint, check the deployed build variables first, not only local files.

## SPA Deep-Link Return Checks

After implementing OAuth callback, recharge return, or any Funloom redirect target in an SPA, verify the deployed deep link directly, not only the homepage:

- Open the callback/return URL shape directly, for example `/funloom/callback?code=test&state=test`.
- Inspect the generated HTML asset URLs. For Vite, relative `base: './'` can make a deep link request `/funloom/assets/...` even when assets are deployed at `/assets/...`.
- Request the referenced JS and CSS URLs and confirm their content types are JavaScript and CSS, not `text/html`.
- If a module script fails with `Expected a JavaScript module script... MIME type "text/html"`, classify it as a game SPA asset routing/build configuration bug unless evidence points elsewhere.

Cloudflare SPA fallback can hide missing asset paths by returning `index.html` with status 200. Content type and first bytes matter more than the HTTP status for this check.

## Cloudflare And Similar Deploy Warnings

When deployment warns that local configuration differs from remote dashboard configuration, pause and explain the impact before continuing. Check routes, zone names, Durable Objects, queues, R2, D1, observability, and secrets. Do not casually overwrite remote config if the warning suggests the local file is stale.

Secrets normally should be managed as platform secrets, not committed into toml/config files.

## Required Manual Configuration

Always separate what the agent can write from what the operator must configure:

- game backend auth secret, such as `JWT_SECRET`
- Funloom app id and app secret for the external game backend
- deployed frontend public Funloom bases
- deployed backend/serverless Funloom secrets
- Funloom platform payment secrets in the Funloom backend only
- correct notify/webhook URLs per environment

Do not list a standard Funloom frontend base as a required manual setting when the OpenAPI host can infer it. If manual configuration is needed, state why the environment is non-standard or which platform mapping is missing.

If payment works in one environment but fails in another, inspect missing platform secrets and notify URL before changing SDK or game code.

## Final Report Requirement

When finishing an integration, state:

- whether verification used local, preview, staging, or production
- which Funloom API/frontend bases were active, without secrets
- which env values were written locally
- which deployed variables or secrets the operator still must set
- whether webhook/payment callback tests require a public URL or deployed environment
