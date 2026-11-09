# Contributing to Aureluis-Ai

Thanks for your interest in improving Aureluis-Ai. This document covers how to get set up, what a good pull request looks like, and the few project-specific rules that matter.

---

## Table of Contents

- [Code of Conduct](#code-of-conduct)
- [Before You Start](#before-you-start)
- [Fork and Clone](#fork-and-clone)
- [Install Dependencies](#install-dependencies)
- [Configure Your Environment](#configure-your-environment)
- [Development Workflow](#development-workflow)
- [Verifying Your Change](#verifying-your-change)
- [Submitting a Pull Request](#submitting-a-pull-request)
- [Project Conventions](#project-conventions)
- [Adding an AI Provider](#adding-an-ai-provider)
- [Reporting Security Issues](#reporting-security-issues)
- [Good First Issues](#good-first-issues)

---

## Code of Conduct

Be respectful and constructive. This project is used by security researchers and learners; harassment, gatekeeping, and "you should already know this" responses are not tolerated. Assume good faith, give credit, and critique the code rather than the person.

Discussions about offensive tooling should stay within authorized, educational, and defensive framing.

---

## Before You Start

**Never commit real credentials.** If your change touches environment handling, use placeholder values like `your_api_key_here`. See [Security Practices](README.md#-security-practices) in the README.

**Keep the scope tight.** A focused pull request that does one thing is far easier to review and far more likely to be merged than a sweeping refactor.

---

## Fork and Clone

```bash
# 1. Fork the repository on GitHub (the "Fork" button, top right)

# 2. Clone your fork
git clone https://github.com/<your-username>/Aureluis-Ai.git
cd Aureluis-Ai

# 3. Add the upstream repository as a remote
git remote add upstream https://github.com/MrVempiree/Aureluis-Ai.git

# 4. Create a branch for your work
git checkout -b feature/short-description
```

Keep your branch up to date with upstream:

```bash
git fetch upstream
git rebase upstream/main
```

---

## Install Dependencies

**Requirements:** Node.js **24** (see `.node-version`) and pnpm **10.26.1**.

```bash
npm install -g pnpm@10.26.1
pnpm install --frozen-lockfile
```

The root `preinstall` script enforces pnpm and refuses to run under npm or yarn. On native Windows, `sh` is unavailable — use Git Bash or WSL, or run `pnpm install --ignore-scripts`.

> **Do not** update dependencies "just in case." `pnpm-lock.yaml` is pinned deliberately, and `pnpm-workspace.yaml` enforces a 24-hour minimum release age as a supply-chain defence. If you genuinely need a version bump, do it in a dedicated PR and say why.

---

## Configure Your Environment

```bash
cp .env.example .env
```

Fill in at minimum:

```bash
TELEGRAM_BOT_TOKEN=your_token_from_botfather
ADMIN_IDS=your_numeric_telegram_id
GEMINI_API_KEY=your_key    # or GROQ_API_KEY / OPENROUTER_API_KEY
PORT=8080
```

Your numeric Telegram ID is **not** your username — get it from [@userinfobot](https://telegram.me/userinfobot).

`.env` is git-ignored. Never force-add it.

---

## Development Workflow

```bash
# API server + Telegram bot (builds, then starts)
pnpm --filter @workspace/api-server run dev

# Component preview sandbox
pnpm --filter @workspace/mockup-sandbox run dev

# Typecheck the whole workspace
pnpm run typecheck

# Build everything
pnpm run build
```

The bot uses long polling, so you can develop against real Telegram without a public URL or webhook. While iterating, you can avoid spamming the live bot by keeping public mode off — only `ADMIN_IDS` can reach a private-mode bot.

### Where things live

| Area | Path |
|------|------|
| Bot startup, command handlers | `artifacts/api-server/src/bot/index.ts` |
| AI providers and fallback | `artifacts/api-server/src/bot/ai.ts` |
| System prompt | `artifacts/api-server/src/bot/systemPrompt.ts` |
| Command menu, welcome/help text | `artifacts/api-server/src/bot/commands.ts` |
| Admin config, user tracking, logs | `artifacts/api-server/src/bot/admin.ts` |
| Sessions | `artifacts/api-server/src/bot/session.ts` |
| Upload handling | `artifacts/api-server/src/bot/fileReader.ts` |
| Code extraction and bundling | `artifacts/api-server/src/bot/codeFiles.ts` |
| Markdown → Telegram HTML | `artifacts/api-server/src/bot/formatter.ts` |
| Express app and routes | `artifacts/api-server/src/app.ts`, `src/routes/` |
| OpenAPI contract | `lib/api-spec/openapi.yaml` |
| Preview sandbox | `artifacts/mockup-sandbox/` |

---

## Verifying Your Change

```bash
pnpm run typecheck
pnpm run build
```

There is currently **no automated test suite** — that is a known gap, and adding one is a genuinely useful contribution. If you add tests, keep the setup minimal; this project deliberately avoids heavyweight frameworks.

If you modify `lib/api-spec/openapi.yaml` or the generated API clients, regenerate them:

```bash
pnpm --filter @workspace/api-spec run codegen
```

If your change touches the OpenAPI contract, update `openapi.yaml` **and** the generated output together, and describe the contract change in your PR description.

---

## Submitting a Pull Request

```bash
git add <your changed files>
git diff --cached --check    # catch whitespace errors
git diff --cached --stat     # review what you're about to commit

git commit -m "Short imperative summary of the change"
git push origin feature/short-description
```

Then open a pull request against `main`.

### What a good PR description contains

- **What** changed and **why**.
- **How** you verified it (commands run and their results).
- **Screenshots** for UI changes in the mockup sandbox.
- **Breaking changes**, especially to environment variables or the API contract.
- Anything you deliberately did **not** do.

### Commit messages

Use the imperative mood: `Add Groq retry backoff`, not `Added` or `adds`. Keep the subject line under ~72 characters.

---

## Project Conventions

- **TypeScript everywhere**, `strict` mode. No `any` in new code without a comment explaining why.
- **2-space indentation**, double quotes, semicolons — matching the existing files.
- **ESM imports** with the `.js` extension in source (`import { logger } from "../lib/logger.js"`), since the API server bundles as ESM.
- **Relative imports inside a package**, `@workspace/*` imports across packages.
- **Keep internal package names** (`@workspace/api-server`, `@workspace/db`, …) as they are. Renaming them is a breaking change with no functional benefit.
- **Comment the *why*, not the *what*.** The code should read clearly without comments restating it.
- **Do not rewrite the system prompt wholesale** in a drive-by change. It is a deliberate product decision, and a rewrite deserves its own PR with discussion.

### Naming

The product name is **Aureluis-Ai**. Use it consistently in source, user-facing strings, generated filenames, docs, and deployment configuration. Don't reintroduce older names, and don't rename third-party technologies or provider model identifiers.

---

## Adding an AI Provider

Providers are defined in `artifacts/api-server/src/bot/ai.ts`. A new provider needs:

1. A client initialized only when its API key is present.
2. A `try<Provider>()` function matching the existing ones, which throws on failure so the fallback loop advances.
3. An entry in the `providers` array in `askAI()` — **order matters**, since that array is the fallback chain.
4. A status flag in `getProviderStatus()` so `/status` reports it.
5. A ping entry in `pingProviders()` so `/ping` tests it.
6. A badge in `providerBadge()` in `bot/index.ts` so replies show which provider answered.
7. A new environment variable in `.env.example`, documented in the README, and added to `render.yaml` as an optional secret.

---

## Reporting Security Issues

**Do not open a public issue for a vulnerability.**

Use GitHub's private vulnerability reporting via the **Security** tab of this repository. Full details, including what is in scope and how to handle sensitive evidence, are in [SECURITY.md](SECURITY.md).

If your change fixes a vulnerability, please describe the issue generically in the PR and disclose the specifics privately to the maintainers.

---

## Good First Issues

Looking for something to start on? These are all real, self-contained improvements:

- **Fix the pre-existing typecheck errors.** `artifacts/api-server/src/bot/index.ts` has `TelegramBot` used as a namespace, an unknown `disable_web_page_preview` option, and an undefined `MAIN_MENU_KEYBOARD`. These break `pnpm run typecheck`.
- **Make `/clearall` actually clear sessions.** The handler acknowledges but does not reset anything. Add a `clearAllSessions()` to `bot/session.ts` and call it.
- **Register admin commands with Telegram.** `setMyCommands` currently lists only public commands, so admin commands are undiscoverable in the Telegram UI.
- **Stream the file download instead of buffering it.** `bot/fileReader.ts` calls `arrayBuffer()` before enforcing the 512 KB limit; enforce the cap while reading so oversized uploads are rejected early.
- **Add a minimal test suite.** Even a handful of `node:test` cases for `parseAdminIds`, `extractCodeFiles`, and `splitMessage` would close a real gap.
- **Wire the database scaffolding.** `lib/db` has no tables and the API never imports it. Defining a schema and a persistence layer for user records would be a substantial but valuable contribution.

---

## Questions?

Open a discussion or an issue. If you want to propose a larger feature, open an issue first to agree on the approach before writing the code — it saves everyone time.

Happy hacking. 🛡️
