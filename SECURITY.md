# Security Policy

Aureluis-Ai is a Telegram AI assistant that forwards user messages to third-party AI providers. It also tracks Telegram user identifiers, usernames and conversation history in process memory, and exposes administrator operations over the Telegram command interface.

## Reporting a Vulnerability

**Please do not open a public GitHub Issue for security reports.**

Report vulnerabilities through GitHub's private vulnerability reporting on this repository:

1. Go to **Security** on the repository page.
2. Select **Advisories** → **Report a vulnerability**.
3. Describe the issue with reproduction steps and impact.

If private vulnerability reporting is not enabled for this repository, open a public issue that contains **only** a short notice that a security report is available, with no technical details, and wait for a maintainer to enable the channel.

Please do not paste live credentials, tokens, or personal data into a report. Redact them and describe where they came from.

## Supported Versions

Security fixes are applied to the current `main` branch. There are no published release tags yet, so only the latest commit on `main` is supported.

| Version | Supported |
| ------- | --------- |
| `main` (latest commit) | ✅ |
| Older commits | ❌ |

## Scope

### In scope

- Authentication or authorization bypass of administrator-only commands (`/stats`, `/users`, `/logs`, `/userlog`, `/ban`, `/unban`, `/broadcast`, `/public`, `/clearall`).
- Hardcoded credentials, leaked tokens, or secrets committed to the repository.
- Missing or bypassable access control that lets a non-administrator reach privileged functionality.
- Unauthenticated access to the Express API surface beyond the intended health endpoint.
- Injection or command execution through uploaded files, uploaded file names, or user-supplied input.
- Resource exhaustion through file uploads or unbounded request bodies.
- Exposure of one user's conversation history or user data to another user.

### Out of scope

- Vulnerabilities in third-party services (Gemini, Groq, OpenRouter, Telegram, or any npm dependency) — report those to the respective maintainers.
- Weaknesses in the AI models themselves, including jailbreaks of the underlying providers.
- Denial of service caused by extreme load against a self-hosted deployment you control.
- Missing hardening that is already covered by the documented threat model in `PRIVACY.md` (for example, in-memory storage being lost on restart).
- Reports that rely on an operator deliberately enabling public mode and then treating public access as a bug.

## Sensitive Evidence Handling

- Do not include real bot tokens, API keys, database URIs, or personal data in a report. Redact them.
- Prefer minimal, synthetic reproductions. A short description of the vulnerable code path is usually enough.
- Do not test against live third-party infrastructure. Reproduce locally, in a controlled lab, or against your own test accounts.
- Do not access data belonging to other users while demonstrating an issue.

## Security Considerations for Operators

If you are self-hosting this project, note the following before exposing it:

- The bot is **private by default**. Administrators are configured via `ADMIN_IDS`; when it is unset, no user can reach privileged commands. Set it explicitly to a comma-separated list of numeric Telegram IDs.
- Administrator commands are gated on the Telegram sender ID. Anyone who can act as one of those IDs can use them.
- `/public on` allows every Telegram user to reach the bot, including the AI providers you have configured. Be aware of the cost and abuse implications before enabling it.
- Uploaded files are read from Telegram and included in AI context. Treat uploads as untrusted input and review them before deploying in a sensitive environment.
- Conversation logs and user records are held in memory and are lost when the process restarts. There is no database persistence.
- Rotate any credential that has ever been exposed publicly. Removing a secret from Git does not invalidate it.

## Responsible Disclosure

Please give maintainers a reasonable opportunity to fix an issue before public disclosure. We appreciate reports that are clear, reproducible and focused on real impact.
