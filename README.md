<div align="center">

# 𝐀𝐔𝐑𝐄𝐋𝐔𝐈𝐒-𝐀𝐈

**An AI-powered Telegram assistant with multi-provider AI integration, code assistance, file analysis and defensive cybersecurity capabilities.**

<sub>Self-hosted Telegram AI bot · AI assistant · chatbot · code generation · Gemini + Groq + OpenRouter fallback · TypeScript monorepo · cybersecurity tooling · open source</sub>

[![GitHub stars](https://img.shields.io/github/stars/MrVempiree/Aureluis-Ai?style=social)](https://github.com/MrVempiree/Aureluis-Ai/stargazers)
[![GitHub forks](https://img.shields.io/github/forks/MrVempiree/Aureluis-Ai?style=social)](https://github.com/MrVempiree/Aureluis-Ai/network/members)
[![GitHub issues](https://img.shields.io/github/issues/MrVempiree/Aureluis-Ai?style=social)](https://github.com/MrVempiree/Aureluis-Ai/issues)
[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![Node.js](https://img.shields.io/badge/Node.js-24-green.svg)](.node-version)
[![pnpm](https://img.shields.io/badge/pnpm-workspace-orange.svg)](pnpm-workspace.yaml)
[![TypeScript](https://img.shields.io/badge/TypeScript-5.9-3178C6.svg)](tsconfig.base.json)
[![Telegram](https://img.shields.io/badge/Telegram-Bot-2AABEE.svg)](https://core.telegram.org/bots)
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg)](CONTRIBUTING.md)
[![Security](https://img.shields.io/badge/Security-Policy-blue.svg)](SECURITY.md)

</div>

---

## 📌 Table of Contents

- [What is Aureluis-Ai](#-what-is-aureluis-ai)
- [Features](#-features)
- [AI Provider Architecture](#-ai-provider-architecture)
- [Commands](#-commands)
- [Administrator Commands](#-administrator-commands)
- [Code Generation & Downloads](#-code-generation--downloads)
- [File Uploads](#-file-uploads)
- [Session Management](#-session-management)
- [API Endpoints](#-api-endpoints)
- [Project Structure](#-project-structure)
- [Tech Stack](#-tech-stack)
- [Installation](#-installation)
- [Environment Configuration](#-environment-configuration)
- [Local Development](#-local-development)
- [Build Commands](#-build-commands)
- [Deployment](#-deployment)
- [Security Practices](#-security-practices)
- [Privacy](#-privacy)
- [Disclaimer](#-disclaimer)
- [Contributing](#-contributing)
- [License](#-license)
- [Known Limitations](#-known-limitations)
- [Troubleshooting](#-troubleshooting)

---

## 💡 What is Aureluis-Ai

**Aureluis-Ai** is a self-hosted Telegram bot that turns any Telegram chat into a multi-provider AI assistant. You talk to it like a person; it replies through Google Gemini, Groq, or OpenRouter — whichever is available — and can hand you back working source code as downloadable files.

It is built for software development, secure coding, and defensive security work: reviewing code, explaining vulnerabilities, writing detection rules, walking through CTF challenges, and triaging suspicious samples.

Everything runs on your own infrastructure. There is no Aureluis-Ai SaaS — you bring your own API keys, your own bot token, and your own server.

---

## ✨ Features

### Core AI
- **Multi-provider fallback** — Gemini → Groq → OpenRouter, automatically, on failure or rate limit.
- **Provider status** (`/status`) — see which API keys are configured.
- **Live provider ping** (`/ping`) — real round-trip latency test against every configured provider.
- **Detailed / concise modes** (`/mode`) — switch response depth per user.
- **Conversation memory** — context is carried across messages within a session.
- **Continue** (`/continue`) — resume a truncated response where it stopped.
- **Clear** (`/clear`) — wipe your conversation history on demand.

### Code & Files
- **Code generation** — full, runnable implementations with imports, error handling and comments.
- **Automatic code-block extraction** — fenced code blocks are detected in every AI reply.
- **Per-file downloads** — each extracted file is sent as a separate Telegram document.
- **Combined JSON bundle** — all generated files packaged into one `aureluis-ai_code.json`.
- **Text & code uploads** — send a file and ask the bot to analyze it.

### Bot Operations
- **Administrator controls** — stats, user list, ban/unban, broadcast, mode toggle.
- **User tracking** — per-user message counts and last-seen timestamps.
- **Ban / unban** — deny specific Telegram IDs access.
- **Broadcast** — push a message to every tracked user.
- **Public / private mode** — private by default (admins only); open it up when you want.
- **Conversation logging** — administrators can review recent exchanges.

### Platform
- **Express 5 health endpoint** — `/api/healthz` for uptime monitors and Render.
- **React + Vite preview sandbox** — a component preview environment in the workspace.
- **Type-safe monorepo** — pnpm workspaces, TypeScript project references, Zod + Orval API codegen.

---

## 🧠 AI Provider Architecture

Aureluis-Ai talks to three providers through two SDKs.

| Provider | SDK | Model | Env var | Role |
|----------|-----|-------|---------|------|
| Google Gemini | `@google/generative-ai` | `gemini-flash-latest` → `gemini-2.0-flash` → `gemini-1.5-flash` | `GEMINI_API_KEY` | Primary |
| Groq | `groq-sdk` | `llama-3.3-70b-versatile` | `GROQ_API_KEY` | Secondary |
| OpenRouter | `openai` (OpenAI-compatible) | `openai/gpt-4o` | `OPENROUTER_API_KEY` | Tertiary |

### Fallback order

```
Gemini  ──fail/rate-limit──▶  Groq  ──fail/rate-limit──▶  OpenRouter
```

- Each provider is tried in turn. A failure or a `429` moves to the next one automatically.
- Rate-limited requests are retried once after a 5-second pause before falling through.
- Gemini tries three model IDs in descending order, so a single retired model does not break the provider.
- Gemini requests are sent with explicit `BLOCK_NONE` thresholds across all four harm categories, as configured in `artifacts/api-server/src/bot/ai.ts`. Review that file before deploying if you need stricter provider-side filtering.

### Token budget

Output length is capped by mode to stay within Groq's free-tier throughput:

| Mode | Max tokens |
|------|-----------|
| `detailed` | 1500 |
| `concise` | 512 |

### Provider status and live ping

`/status` reports which keys are present. `/ping` sends a real one-word request to every provider and returns online/offline plus latency in milliseconds.

---

## 💬 Commands

### Public commands

| Command | Description |
|---------|-------------|
| `/start` | Welcome message and capability overview |
| `/help` | Full command list |
| `/status` | Which AI providers are configured |
| `/ping` | Live latency test across all providers |
| `/code` | Write code in any language |
| `/malware` | Malware analysis, detection rules and defensive research |
| `/phishpage` | Phishing-awareness demonstrations and analysis |
| `/hacktools` | Authorized security testing and defensive tooling |
| `/findgroups` | Legitimate security communities and public resources |
| `/scam` | Scam detection, red flags and awareness education |
| `/leaks` | Breach analysis and defensive incident response |
| `/learn` | Programming and security education |
| `/sourcecode` | Educational and defensive source references |
| `/autoscript` | Defensive automation, log analysis and monitoring scripts |
| `/analyze` | Analyze code, files or suspicious samples |
| `/scan` | Authorized security assessment guidance |
| `/exploit` | Vulnerability education and controlled-lab reproduction |
| `/obfuscate` | Analyze and deobfuscate code |
| `/ctf` | Solve legitimate CTF challenges |
| `/resources` | Reliable public learning resources |
| `/continue` | Continue the previous response |
| `/clear` | Clear conversation history |
| `/mode` | Switch between `detailed` and `concise` |

Every command is also registered with Telegram's command menu via `setMyCommands`.

### Administrator commands

| Command | Description |
|---------|-------------|
| `/adminhelp` | Admin command list |
| `/stats` | User counts, message totals, uptime, access mode |
| `/users` | Tracked users with message counts and last-seen |
| `/logs` | Recent conversations |
| `/userlog <id>` | Conversation log for one user |
| `/ban <id>` | Ban a Telegram user ID |
| `/unban <id>` | Unban a Telegram user ID |
| `/broadcast <msg>` | Send a message to all tracked users |
| `/boton` | Enable public mode |
| `/botoff` | Return to private (admin-only) mode |
| `/clearall` | Clear sessions |

**Access control:** every administrator command checks the sender's Telegram numeric ID against the `ADMIN_IDS` list. A user cannot invoke them, and a user cannot promote themselves — `/addadmin` exists only as an administrator operation and requires an existing administrator to call it.

---

## 💻 Code Generation & Downloads

When the AI replies with fenced code blocks, Aureluis-Ai automatically:

1. Extracts every ` ```lang ... ``` ` block from the response.
2. Maps the language tag to a file extension (40+ languages supported: C, C++, Rust, Go, Python, Bash, PowerShell, JavaScript, TypeScript, PHP, Java, Kotlin, Swift, SQL, YAML, Assembly, and more).
3. Sends each file as its own Telegram document with a caption showing filename, language and size.
4. Bundles all files into a single JSON document named **`aureluis-ai_code.json`**, containing `generated_by`, a timestamp, file count, and each file's name, language, size and source.

Multiple files of the same language are numbered (`code.py`, `code_2.py`) so nothing is overwritten.

---

## 📂 File Uploads

Send a document to the bot and it is added to your next AI request as context.

**Supported for text extraction:** `.txt`, `.py`, `.js`, `.ts`, `.c`, `.cpp`, `.cs`, `.java`, `.rb`, `.go`, `.rs`, `.swift`, `.kt`, `.php`, `.pl`, `.lua`, `.sh`, `.bash`, `.ps1`, `.html`, `.css`, `.scss`, `.json`, `.yaml`, `.yml`, `.toml`, `.xml`, `.sql`, `.md`, `.asm`, `.s`, `.sol`, `.dart`, `.zig`, `.nim`, `.jl`, `.r`, `.m`, `Makefile`, `Dockerfile`, `.gitignore`, `.log`, `.csv`, `.tsv`, and matching MIME types.

**Size limit:** 512 KB.

**Binary files** (EXE, DLL, ZIP, images, and so on) are **not** analyzed as binary content. The bot receives a metadata notice based on the filename and MIME type only. There is no binary disassembly or sample-hashing pipeline in this build.

Uploaded content is treated as **data to analyze**, never as instructions to execute.

---

## 🧠 Session Management

- Sessions live in memory, keyed by Telegram user ID.
- A session expires after **2 hours of inactivity** (swept every 30 minutes).
- Conversation history keeps the **latest 40 messages**.
- `/clear` resets your session immediately.
- `/mode` state is per-user and persists for the life of the session.

**All session data is lost when the process restarts.**

---

## 🌐 API Endpoints

The Express 5 server exposes one public route under the `/api` prefix.

### `GET /api/healthz`

```json
{ "status": "healthy" }
```

Used by Render's health check and suitable for uptime monitoring. The route contract is defined in `lib/api-spec/openapi.yaml` and the generated Zod/Orval clients live under `lib/api-zod` and `lib/api-client-react`.

There are no other public API routes, no authentication middleware, and no database-backed endpoints.

---

## 📁 Project Structure

```
Aureluis-Ai/
├── artifacts/
│   ├── api-server/              # The Telegram bot + Express API
│   │   ├── src/
│   │   │   ├── index.ts         # Entry point: starts bot + HTTP server
│   │   │   ├── app.ts           # Express app, middleware, routes
│   │   │   ├── routes/          # health.ts, index.ts
│   │   │   ├── bot/
│   │   │   │   ├── index.ts     # Bot startup, command + upload handlers
│   │   │   │   ├── ai.ts        # Provider clients, fallback, ping
│   │   │   │   ├── systemPrompt.ts
│   │   │   │   ├── commands.ts  # Command menu, welcome/help text
│   │   │   │   ├── admin.ts     # Admin config, user tracking, logs
│   │   │   │   ├── session.ts   # In-memory sessions
│   │   │   │   ├── fileReader.ts # Upload handling
│   │   │   │   ├── codeFiles.ts # Code extraction + JSON bundle
│   │   │   │   ├── formatter.ts # Markdown → Telegram HTML
│   │   │   │   └── emojis.ts    # Custom emoji IDs
│   │   │   └── lib/logger.ts    # Pino logger
│   │   └── build.mjs            # esbuild bundler
│   └── mockup-sandbox/          # React + Vite component preview
├── lib/
│   ├── api-spec/                # OpenAPI spec + Orval codegen config
│   ├── api-zod/                 # Generated Zod schemas
│   ├── api-client-react/        # Generated React API client
│   └── db/                      # PostgreSQL + Drizzle scaffolding
├── scripts/                     # Workspace utility scripts
├── .env.example                 # Safe placeholder configuration
├── render.yaml                  # Render deployment blueprint
├── pnpm-workspace.yaml          # Workspace + supply-chain policy
└── package.json                 # Root workspace scripts
```

---

## 🛠 Tech Stack

| Layer | Technology |
|-------|-----------|
| Runtime | Node.js 24 |
| Package manager | pnpm 10.26.1 (workspace) |
| Language | TypeScript 5.9 |
| API | Express 5 |
| Telegram | `node-telegram-bot-api` (long polling) |
| AI — Gemini | `@google/generative-ai` |
| AI — Groq | `groq-sdk` |
| AI — OpenRouter | `openai` SDK (OpenAI-compatible endpoint) |
| Logging | Pino + pino-http + pino-pretty |
| Validation | Zod |
| API codegen | Orval (from `openapi.yaml`) |
| Database | PostgreSQL + Drizzle ORM *(scaffolding only)* |
| UI sandbox | React 19, Vite 7, Tailwind CSS 4, Radix UI, shadcn/ui |
| Bundler | esbuild |

---

## 📥 Installation

### Requirements

- **Node.js 24** (see `.node-version`)
- **pnpm 10.26.1** — `npm install -g pnpm@10.26.1`
- A **Telegram bot token** from [@BotFather](https://core.telegram.org/bots)
- At least one **AI provider API key**
- Your **numeric Telegram user ID** for administrator access

> The root `preinstall` script enforces pnpm and refuses to run under npm or yarn. On Windows, run it through Git Bash or WSL, or install with `pnpm install --ignore-scripts`.

### Clone and install

```bash
git clone https://github.com/MrVempiree/Aureluis-Ai.git
cd Aureluis-Ai
pnpm install --frozen-lockfile
```

---

## 🔧 Environment Configuration

Copy the example file and fill in your values:

```bash
cp .env.example .env
```

| Variable | Required | Description |
|----------|----------|-------------|
| `TELEGRAM_BOT_TOKEN` | ✅ | Bot token from @BotFather. Without it the API starts but the bot does not. |
| `ADMIN_IDS` | ✅ for admin access | Comma-separated numeric Telegram user IDs, e.g. `123456789,987654321`. Unset ⇒ **no one** can use admin commands. |
| `GEMINI_API_KEY` | optional | Google Gemini key. |
| `GROQ_API_KEY` | optional | Groq key. |
| `OPENROUTER_API_KEY` | optional | OpenRouter key. |
| `PORT` | ✅ | HTTP port. Defaults to `8080` in local runs; Render sets `10000`. |
| `NODE_ENV` | optional | `production` in deployed environments. |

At least one AI provider key is required for the bot to answer anything.

Find your Telegram numeric ID by messaging [@userinfobot](https://telegram.me/userinfobot).

**Never commit `.env`.** It is already listed in `.gitignore`, along with `.env.*` and `.env.txt`.

---

## 🧑‍💻 Local Development

```bash
# Terminal 1 — API server + Telegram bot (builds first, then starts)
pnpm --filter @workspace/api-server run dev

# Terminal 2 — component preview sandbox
pnpm --filter @workspace/mockup-sandbox run dev

# Terminal 3 — typecheck everything
pnpm run typecheck
```

The bot uses **long polling**, so no public URL or webhook is required for local development.

---

## 🏗 Build Commands

```bash
pnpm install --frozen-lockfile    # install exactly what the lockfile pins
pnpm run typecheck                # typecheck all workspace packages
pnpm run build                    # typecheck + build every package
pnpm --filter @workspace/api-server run build      # bundle the API server
pnpm --filter @workspace/mockup-sandbox run build  # build the preview sandbox
pnpm --filter @workspace/api-spec run codegen      # regenerate API client (only after editing openapi.yaml)
```

> **Note on the sandbox build:** `pnpm-workspace.yaml` deliberately excludes the Windows Rollup binary to keep the lockfile Linux/Replit-only. The mockup sandbox therefore builds on Linux/macOS but not natively on Windows. The API server bundles with esbuild and builds everywhere.

---

## 🚀 Deployment

### Render (blueprint included)

`render.yaml` defines a Node web service named **`aureluis-ai`**:

```yaml
buildCommand: npm install -g pnpm@10.26.1 && pnpm install --frozen-lockfile && pnpm --filter @workspace/api-server run build
startCommand: pnpm --filter @workspace/api-server run start
healthCheckPath: /api/healthz
```

Set these in the Render dashboard (or as encrypted secrets):

- `TELEGRAM_BOT_TOKEN` — required
- `ADMIN_IDS` — required for admin access
- `GEMINI_API_KEY`, `GROQ_API_KEY`, `OPENROUTER_API_KEY` — at least one

`PORT` is set to `10000` by the blueprint.

### Other platforms

Any host that runs Node 24 and can run a background process works:

```bash
pnpm install --frozen-lockfile
pnpm --filter @workspace/api-server run build
pnpm --filter @workspace/api-server run start
```

The bot starts polling as soon as `TELEGRAM_BOT_TOKEN` is present. Without it, the HTTP server still runs and `/api/healthz` still responds — the bot simply is not started.

---

## 🔐 Security Practices

This project is designed to be self-hosted, so **you** are the operator of record.

- **No secrets in the repository.** `.env`, `.env.*` and `.env.txt` are git-ignored. Only placeholder values live in `.env.example`.
- **No hardcoded administrator IDs.** Admins come from `ADMIN_IDS`. If it is unset, admin commands are inaccessible — the system fails closed.
- **Every admin command is authorized.** `/stats`, `/users`, `/logs`, `/userlog`, `/ban`, `/unban`, `/broadcast`, `/boton`, `/botoff` and `/clearall` all check the sender ID before acting. Privilege escalation is not available to regular users.
- **Private by default.** The bot is admin-only until `/boton` is explicitly issued.
- **Banned users are denied**, including in public mode.
- **Provider thresholds are explicit.** Gemini is configured with `BLOCK_NONE` in `artifacts/api-server/src/bot/ai.ts`; operators who need stricter filtering should adjust that file themselves.
- **Uploads are bounded** to 512 KB and treated as untrusted data, never as executable content.
- **Supply-chain hardening.** `pnpm-workspace.yaml` enforces a 24-hour minimum release age for npm packages, pins the esbuild version, and strips non-target platform binaries.
- **Dependency pin.** `pnpm install --frozen-lockfile` guarantees the reviewed dependency graph is what gets installed.

**If a credential was ever exposed publicly, rotate it.** Deleting a secret from Git does not invalidate it — the old value remains valid until the provider issues a new one.

---

## 🔒 Privacy

See [PRIVACY.md](PRIVACY.md) for the full policy.

In short: the bot stores Telegram IDs, usernames, first names, message counts, and conversation history **in memory only**. Everything is lost on restart. Prompts and context are sent to the configured AI providers, which process them under their own policies.

---

## ⚠️ Disclaimer

See [DISCLAIMER.md](DISCLAIMER.md) for the full text.

> Aureluis-Ai is provided for lawful educational, software development, defensive security and authorized testing purposes.
>
> Users must only interact with systems, services, networks and information they own or have explicit permission to access.
>
> Any illegal, harmful, fraudulent, unauthorized or abusive use is strictly prohibited. Users are solely responsible for their conduct, configuration, deployment and compliance with applicable law.
>
> The maintainers do not authorize, encourage or endorse illegal activity.
>
> The software is provided **AS IS** and **AS AVAILABLE**, without warranties of any kind, to the maximum extent permitted by law.
>
> The maintainers do not guarantee accuracy, uninterrupted service, suitability, security against every threat or specific outcomes.
>
> The user assumes responsibility for lawful use. Liability is disclaimed to the fullest extent legally permitted, without excluding rights or liabilities that cannot lawfully be excluded.

**A disclaimer does not grant permission to perform unauthorized actions, and does not remove legal responsibility from the operator.**

---

## 🤝 Contributing

Contributions are welcome — see [CONTRIBUTING.md](CONTRIBUTING.md).

```bash
git clone https://github.com/MrVempiree/Aureluis-Ai.git
cd Aureluis-Ai
pnpm install --frozen-lockfile
pnpm run typecheck
```

Please keep new AI provider integrations behind the same fallback interface in `artifacts/api-server/src/bot/ai.ts`, and never commit real credentials.

---

## 📄 License

Released under the [MIT License](LICENSE).

```
MIT License

Copyright (c) 2026 Lavish Gangwar

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
```

Third-party components retain their own licenses — notably the shadcn/ui components in `artifacts/mockup-sandbox` (MIT, © shadcn) and Radix UI primitives (MIT, © WorkOS). Their copyright is not replaced by the project owner.

---

## ⚠️ Known Limitations

Being explicit about what this build does **not** do:

- **No persistent storage.** Sessions, user records, bans, and conversation logs are all in-memory and vanish on restart. The `lib/db` package is Drizzle/PostgreSQL **scaffolding only** — the API server never imports it, and there are no tables defined yet.
- **No database is required.** `DATABASE_URL` is not read by the running application. `lib/db` throws if imported without it, but nothing in the live import graph touches it.
- **Binary uploads are not analyzed.** Only the filename and MIME type are surfaced. No disassembly, no hashing, no sandboxing.
- **The mockup sandbox is a component preview environment**, not a customer-facing chat UI. Its generated component map (`src/.generated/mockup-components.ts`) is currently empty, so `/preview/<name>` has nothing to render until components are registered.
- **`/clearall` does not actually clear sessions.** The handler acknowledges the command but does not yet call a session-reset routine. Use `/clear` per user until this is implemented.
- **`/addadmin` is not in the command menu.** It exists as an admin-only handler that adds an ID to the runtime set, but it resets on restart. Add admins to `ADMIN_IDS` instead.
- **The Telegram command menu does not list admin commands.** Admin commands are reachable by typing them, but only the public commands are registered via `setMyCommands`.
- **No automated test suite.** The project ships no `test` script. Verification is `pnpm run typecheck` and `pnpm run build`.
- **The API has a single route** and no authentication layer.
- **No webhook support** — long polling only.
- **Rate limits follow the provider.** Free-tier Gemini and Groq keys will throttle under load; the bot falls back automatically but may still return a rate-limit message.
- **Typecheck currently reports pre-existing errors** in `artifacts/api-server/src/bot/index.ts` (TelegramBot used as a namespace, an unknown `disable_web_page_preview` option, and an undefined `MAIN_MENU_KEYBOARD`). These are unrelated to the rename and are left as-is. The esbuild production bundle — which is what Render actually deploys — builds successfully.

---

## 🔧 Troubleshooting

**The bot does not start.**
Check `TELEGRAM_BOT_TOKEN`. Without it the API server still runs and `/api/healthz` responds, but no bot is created. A log line reads `TELEGRAM_BOT_TOKEN not set — bot will not start`.

**The bot responds but says no providers are online.**
At least one of `GEMINI_API_KEY`, `GROQ_API_KEY`, `OPENROUTER_API_KEY` must be set and valid. Use `/status` to confirm detection, then `/ping` for a live test.

**`/ping` shows a rate-limit error.**
Free tiers throttle. Wait about 60 seconds and retry. Aureluis-Ai already retries once after a 5-second pause before falling through to the next provider.

**I am not recognized as an administrator.**
`ADMIN_IDS` must contain your **numeric** Telegram ID — not your username. It is comma-separated, no spaces needed. If it is unset or invalid, no administrator commands work at all. Restart the process after changing it.

**"You are banned from this bot."**
Ask an administrator to run `/unban <your numeric id>`.

**Another user cannot reach the bot.**
The bot is private by default. An administrator must run `/boton` to enable public mode.

**`pnpm install` fails with `ERR_PNPM_LIFECYCLE_SCRIPT_FAILED`.**
The root `preinstall` script shells out to `sh`, which is not present on native Windows. Use Git Bash or WSL, or run `pnpm install --ignore-scripts`.

**The mockup sandbox build fails with `Cannot find module '@rollup/rollup-win32-x64-msvc'`.**
Expected on Windows — `pnpm-workspace.yaml` excludes the Windows Rollup binary. Build the sandbox on Linux/macOS, or build only the API server with `pnpm --filter @workspace/api-server run build`.

**The mockup sandbox shows "No component found".**
Its generated component map is empty. Register components under `src/components/mockups/` and regenerate `src/.generated/mockup-components.ts`.

**The API server will not start.**
`PORT` is required. The process throws at boot if it is missing or not a positive number.

**Bot messages are not formatted.**
Telegram rejects malformed HTML, so the bot automatically retries as plain text. If formatting looks wrong, check the AI output for unbalanced tags.

---

<div align="center">

**Made with ☕ by Lavish Gangwar**

If Aureluis-Ai helped you, consider starring the repo — it genuinely helps other people find it.

</div>
