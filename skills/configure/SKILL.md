---
name: configure
description: Set up the Lark/Feishu channel — save App ID and App Secret, check channel status, guide through app creation. Use when the user pastes credentials, asks to configure Lark/Feishu, or wants to check channel status.
user-invocable: true
allowed-tools:
  - Read
  - Write
  - Bash(ls *)
  - Bash(mkdir *)
---

# /lark:configure — Lark/Feishu Channel Setup

Writes App ID and App Secret to `~/.claude/channels/lark/.env` and orients the user on access policy. The server reads both files at boot.

Arguments passed: `$ARGUMENTS`

---

## Dispatch on arguments

### No args — status and guidance

Read both state files and give the user a complete picture:

1. **Credentials** — check `~/.claude/channels/lark/.env` for `LARK_APP_ID` and `LARK_APP_SECRET`. Show set/not-set; if set, show masked values (`cli_xxx...`).

2. **Domain** — check for `LARK_DOMAIN` in the env file. Show `feishu` (default) or `lark`.

3. **Access** — read `~/.claude/channels/lark/access.json` (missing = defaults). Show:
   - DM policy and what it means
   - Allowed chats: count and IDs
   - Groups: count with chat_ids

4. **What next** — end with a concrete next step:
   - No credentials → *"Run `/lark:configure <APP_ID> <APP_SECRET>` with credentials from the Lark Developer Console."*
   - Credentials set, nobody allowed → *"Send your bot a DM in Lark. Your chat will be auto-added (open policy). Or run `/lark:access allow <chat_id>` directly."*
   - Credentials set, someone allowed → *"Ready. DM your bot to reach the assistant."*

**Push toward lockdown.** `open` policy is for initial setup. Once you have your chat_id, switch to `allowlist`:
1. Tell the user their allowFrom list.
2. Ask: *"Is that everyone who should reach you through this bot?"*
3. If yes and policy is `open` → offer to run `/lark:access policy allowlist`.
4. If no → *"Send a DM to capture your chat_id, then we'll lock it."*

### `<APP_ID> <APP_SECRET>` — save credentials

1. Parse: first token is APP_ID, second is APP_SECRET.
2. `mkdir -p ~/.claude/channels/lark`
3. Read existing `.env` if present; update/add both keys, preserve other keys.
4. Write back, no quotes around values.
5. Confirm, then show no-args status.

### `<APP_ID> <APP_SECRET> lark` — save with Lark (international) domain

Same as above but also write `LARK_DOMAIN=lark`.

### `domain <feishu|lark>` — change domain

1. Read `.env`, update/add `LARK_DOMAIN=<value>`, write back.
2. Confirm. Note: server needs restart.

### `clear` — remove credentials

Delete `LARK_APP_ID=` and `LARK_APP_SECRET=` lines. Warn the user their channel will stop working.

---

## App setup guide (show when no credentials are set)

Walk the user through creating a Lark/Feishu app:

**For Feishu (飞书):**
1. Open [Feishu Developer Console](https://open.feishu.cn/app)
2. Click **Create App** → **Self-Built App**
3. Fill in name and description
4. Go to **Capabilities** → enable **Bot**
5. Go to **Event Subscriptions** → set subscription mode to **Long Connection** (no webhook URL needed)
6. Subscribe to event: `im.message.receive_v1`
7. Go to **Permissions** → enable:
   - `im:message` (read/write messages)
   - `im:message.receive_v1` (receive messages)
   - `im:resource` (for file/image uploads)
8. Go to **App Credentials** → copy App ID and App Secret
9. Publish the app (or add it to your workspace for testing)

**For Lark (international):**
- Same steps at [Lark Developer Console](https://open.larksuite.com/app)
- Pass `lark` as the third argument: `/lark:configure <APP_ID> <APP_SECRET> lark`

**Adding the bot to a workspace:**
- Feishu: App page → **Availability** → enable for your org or specific users
- Lark: Same under **Availability**

---

## Implementation notes

- The channels dir might not exist if the server hasn't run yet. Missing file = not configured.
- The server reads `.env` once at boot. Credential changes need a session restart or `/reload-plugins`.
- `access.json` is re-read on every inbound message — policy changes take effect immediately.
- App ID format: `cli_xxxxxxxxxxxxxx` (Feishu) or similar for Lark.
