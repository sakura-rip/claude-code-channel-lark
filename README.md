# Lark / Feishu

Connect a Lark or Feishu bot to your Claude Code with an MCP server.

The MCP server connects to Lark using **WSClient long connection mode** — no webhook or public URL needed. When you message the bot, the server forwards the message to your Claude Code session. Claude replies through the Lark API.

Supports both [Feishu](https://www.feishu.cn) (China) and [Lark](https://www.larksuite.com) (international).

## Prerequisites

- [Bun](https://bun.sh) — the MCP server runs on Bun. Install with `curl -fsSL https://bun.sh/install | bash`.
- A Lark/Feishu self-built app with bot capability enabled.

## Quick Setup

**1. Create a Lark/Feishu app.**

**Feishu:** Open [open.feishu.cn/app](https://open.feishu.cn/app) → **Create App** → **Self-Built App**.

**Lark:** Open [open.larksuite.com/app](https://open.larksuite.com/app) → same steps.

In your app:
- **Capabilities** → enable **Bot**
- **Event Subscriptions** → set mode to **Long Connection** (no webhook URL needed)
- Subscribe to: `im.message.receive_v1`
- **Permissions** → enable `im:message`, `im:message.receive_v1`, `im:resource`
- **App Credentials** → copy your App ID and App Secret
- **Availability** → publish or add to your workspace

**2. Install the plugin.**

Add the marketplace and install:

```
/plugin marketplace add padimin/claude-code-channel-lark
/plugin install claude-channel-lark@claude-channel-lark
```

**3. Save your credentials.**

```
/lark:configure cli_xxxxxx YourAppSecret
```

For Lark (international, not Feishu):
```
/lark:configure cli_xxxxxx YourAppSecret lark
```

This writes credentials to `~/.claude/channels/lark/.env`.

**4. Relaunch with the channel flag.**

Channels are in research preview. Custom channels require the development flag:

```sh
claude --dangerously-load-development-channels plugin:claude-channel-lark@claude-channel-lark
```

**5. Send yourself a message.**

DM your bot on Lark/Feishu. With the default `open` policy, your chat is auto-added to the allowlist on first contact. Your next message reaches the assistant.

**6. Lock it down.**

Once you're set up, switch to `allowlist` so other users can't reach the bot:

```
/lark:access policy allowlist
```

## Access control

State lives in `~/.claude/channels/lark/access.json`.

| Policy | Behavior |
|--------|----------|
| `open` | Any DM is delivered; chat_id auto-added on first contact. Good for personal use. |
| `allowlist` | Only chat_ids in `allowFrom` get through. Recommended after setup. |
| `disabled` | No DMs accepted. |

**Skill commands:**

| Command | Effect |
|---------|--------|
| `/lark:access` | Show current status |
| `/lark:access allow <chat_id>` | Add a DM chat to allowlist |
| `/lark:access remove <chat_id>` | Remove from allowlist |
| `/lark:access policy allowlist` | Switch to allowlist mode |
| `/lark:access group add <chat_id>` | Allow a group chat (requires @mention by default) |
| `/lark:access group add <chat_id> --no-mention` | Allow group without @mention requirement |
| `/lark:access group rm <chat_id>` | Remove a group |

**Getting a chat_id:** Send the bot a DM while policy is `open` — the server logs the chat_id. Or check the "not authorized" message the bot sends when policy is `allowlist`.

## Tools exposed to the assistant

| Tool | Purpose |
|------|---------|
| `reply` | Send a text message. Takes `chat_id` + `text`, optionally `reply_to` (message_id) for threading and `files` (absolute paths) for attachments. Auto-chunks long text. |
| `reply_card` | Send a markdown card. Takes `chat_id` + `content` (markdown), optionally `title`, `color`, and `reply_to`. Renders code blocks, lists, tables. |

Images (`.jpg`/`.png`/`.gif`/`.webp`) upload as Lark image messages with inline preview. Other files upload as Lark file messages.

## Environment variables

Set in `~/.claude/channels/lark/.env`:

| Variable | Description |
|----------|-------------|
| `LARK_APP_ID` | Your app's App ID (`cli_xxx`) |
| `LARK_APP_SECRET` | Your app's App Secret |
| `LARK_DOMAIN` | `feishu` (default) or `lark` for international |

Shell environment variables take precedence over the file.
