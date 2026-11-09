# Privacy Policy

_Last updated: 9 November 2026_

Aureluis-Ai is a **self-hosted** Telegram bot. The person or organisation that deploys it is the operator and the data controller. If you are running your own instance, you are responsible for how it is configured and for the data it processes.

This policy describes what the **software** does by default. It does not describe what any particular deployment does — an operator can and should add their own logging, analytics or persistence.

---

## What data is collected

When you interact with the bot, the following is handled:

| Data | Where it is held | Why |
|------|------------------|-----|
| Telegram numeric user ID | In-memory user map | Identify you, apply bans, count you in stats |
| Telegram username | In-memory user map | Display a readable name in admin user lists |
| Telegram first name | In-memory user map | Display a readable name in admin user lists |
| Message count and last-seen timestamp | In-memory user map | `/stats` and `/users` |
| Your messages and the bot's replies | In-memory session + conversation log | Maintain conversation context; admin review |
| Broadcast messages you receive | In-memory broadcast log | Record what was sent |
| Uploaded file names, MIME types and text content | Sent to the AI provider for that request | Answer questions about the file |

---

## What is stored in memory

Everything. Aureluis-Ai has **no database**.

- **User records** — Telegram ID, username, first name, message count, last seen.
- **Ban list** — banned Telegram IDs.
- **Conversation sessions** — the last 40 messages per user, expiring after 2 hours of inactivity.
- **Conversation logs** — up to **200** question/answer pairs, accessible to administrators.
- **Broadcast history** — up to **50** broadcast records; `/logs`-style views show the most recent few.

All of it lives in the Node.js process's memory.

---

## What is lost on restart

**Everything above.** A redeploy, crash, or container restart wipes:

- all user tracking records,
- all bans and unbans,
- all conversation sessions and history,
- the entire administrator conversation log,
- broadcast history.

This is a deliberate design property of the current build, not a bug that can be worked around. If you need persistence, you would have to add a database layer — the `lib/db` package in this repository is scaffolding for exactly that and is not wired into the running application.

---

## Conversation-log retention

- A conversation log entry is created for each AI exchange, containing the question, the answer, the provider that served it, and the timestamp.
- The log keeps a rolling window of **200 entries**; the oldest is discarded as new entries arrive.
- The log is readable by administrators through `/logs` and `/userlog <id>`.
- Any user can delete their own conversation context with `/clear`. Note that this clears the **session**; entries already written to the administrator conversation log are not retroactively removed.

---

## Administrator access

Administrators are Telegram users whose numeric IDs appear in the `ADMIN_IDS` environment variable. They can:

- view tracked users and their message counts,
- read recent conversations and per-user conversation history,
- ban and unban users,
- broadcast messages to all tracked users,
- toggle public/private mode.

Any administrator can read the conversations of any other user. Treat administrator access as equivalent to access to the full conversation history of your deployment.

---

## External AI provider processing

When you send a message, it is forwarded to the AI provider that serves the request. Aureluis-Ai supports:

- **Google Gemini**
- **Groq**
- **OpenRouter**

Depending on which key is configured and whether an earlier provider failed, **your message text, your uploaded file contents, and recent conversation context are transmitted to one of these providers** — in some cases to more than one, because the bot may try a provider, receive a failure, and then fall through to the next.

Consequences you should understand:

- These are **third-party services**. Their own privacy policies and terms govern what they do with the data, including possible retention for model training depending on your account and plan.
- Message content is **not** end-to-end encrypted and is **not** anonymised before sending. The provider sees the Telegram user ID in some request paths and the full text of your context in all of them.
- If you are sending sensitive material, check your provider's data-handling terms first, or point the bot at a provider you have configured for that purpose.

---

## Data deletion limitations

Because storage is in-memory, deletion is mostly automatic:

| Action | Effect |
|--------|--------|
| `/clear` | Removes that user's session history |
| Process restart / redeploy | Removes **all** user data, logs, bans and sessions |
| `/unban <id>` | Removes the ban entry |

What cannot be done from inside the bot:

- **Selective deletion of a single conversation-log entry.** The log is append-only with a rolling window.
- **Deletion of data already held by an AI provider.** Once a prompt has been sent, you must use that provider's own mechanisms to request deletion.
- **Deletion from Telegram.** Telegram holds its own copies of messages and bot interactions.

---

## What is NOT collected

Aureluis-Ai does **not**:

- collect email addresses, phone numbers, or payment details,
- read your contacts, location, or Telegram profile photo,
- track you across websites or build an advertising profile,
- use third-party analytics, advertising or tracking pixels,
- persist anything to disk or to an external database.

---

## Operator obligations

If you deploy Aureluis-Ai, you are the operator and you are responsible for:

- Disclosing this behaviour to your users.
- Complying with applicable data-protection law for your jurisdiction, including GDPR obligations if you are serving EU users.
- Configuring administrator access narrowly — `ADMIN_IDS` should contain only people who genuinely need it.
- Not enabling public mode without understanding the exposure that creates.
- Publishing your own contact route for privacy requests, since this repository cannot receive them on your behalf.

---

## Changes to this policy

This policy describes the behaviour of the software as shipped in this repository. If you upgrade Aureluis-Ai, re-read this file — upstream changes to storage or provider behaviour will change what applies.

## Contact

For privacy questions about **this repository and this software**, open an issue on the repository. For questions about a **specific deployment**, contact that deployment's operator — they are the data controller and this project has no access to their instance.
