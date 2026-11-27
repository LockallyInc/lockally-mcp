<p align="center"><img src="icon.png" width="96" height="96" alt="Lockally"></p>

# Lockally MCP server

An email inbox for your AI agent. Connect a Lockally mailbox to ChatGPT, Claude, Cursor or any MCP client, and the assistant can read whole conversations, triage the inbox, and reply in-thread on your own domain, with replies held for your approval until you say send.

Most email tools let an assistant send. This one lets it receive.

**Now in the ChatGPT plugin directory:** [add Lockally to ChatGPT](https://chatgpt.com/plugins/plugin_asdk_app_6af5574ad81c8191affe0f19cf37004c).

```text
Server URL:  https://mcp.lockally.com/
Transport:   Streamable HTTP
Sign-in:     OAuth (recommended), or a Lockally API key as a Bearer token
Registry:    com.lockally/email  (official MCP Registry)
```

This repository holds the connection guides, the registry entry, an agent skill and examples. The server itself is hosted by Lockally; there is nothing to install or run.

## What you need

- A Lockally account with a workspace and at least one mailbox on your own domain. Any plan, including Free. Sign up at [lockally.com](https://lockally.com).
- To sign in with OAuth: the email and password you use for Lockally webmail or the dashboard, plus your two-step code if you have one switched on.
- Or, for clients that take a header instead of OAuth: a Lockally API key with the `inboxes:read` scope, plus `inboxes:write` if the assistant should be able to reply. Create one in the dashboard under API Keys and grant it the mailboxes it may use.

## Connect

### Claude (claude.ai, Desktop, mobile)

Settings, then Connectors, then **Add custom connector**. Enter `https://mcp.lockally.com/` and sign in with your Lockally account when prompted. No client ID or secret is needed: Claude registers itself through our Client ID Metadata Document support.

### ChatGPT

Add Lockally from the ChatGPT plugin directory: open [Lockally in ChatGPT](https://chatgpt.com/plugins/plugin_asdk_app_6af5574ad81c8191affe0f19cf37004c), add it, and sign in with your Lockally account when prompted. Then choose which mailboxes ChatGPT may use.

Or add a custom MCP server yourself with the URL `https://mcp.lockally.com/` and choose OAuth.

### Claude Code

```bash
claude mcp add --transport http lockally https://mcp.lockally.com/
```

Then run `/mcp` inside Claude Code and choose Lockally to sign in.

### Cursor

`~/.cursor/mcp.json` (see [`examples/cursor-mcp.json`](examples/cursor-mcp.json)):

```json
{
  "mcpServers": {
    "lockally": {
      "url": "https://mcp.lockally.com/",
      "headers": { "Authorization": "Bearer lk_live_your_key" }
    }
  }
}
```

### VS Code

`.vscode/mcp.json` (see [`examples/vscode-mcp.json`](examples/vscode-mcp.json)). VS Code prompts for the key and keeps it out of the file.

### Any other client

Point it at `https://mcp.lockally.com/` over Streamable HTTP. An unauthenticated request returns `401` with a `WWW-Authenticate` header naming `https://mcp.lockally.com/.well-known/oauth-protected-resource`, which points at the authorization server `https://api.lockally.com` (OAuth 2.1, PKCE S256, Client ID Metadata Documents, public clients). Clients without OAuth can send `Authorization: Bearer <Lockally API key>` instead.

## What a connection can reach

- **Signed in with OAuth:** only your own mailbox and the shared mailboxes you are a member of, and you choose which of those when you connect. An administrator cannot use their own sign-in to connect other people's mailboxes. Workspace administrators can switch off member connections and revoke any connection under Third-party Apps in the dashboard; you can disconnect your own in webmail under Settings, Connected accounts.
- **With an API key:** only the mailboxes granted to that key. Organisation-wide access, such as a support bot reading support@, uses an API key, created deliberately in the dashboard and recorded in the audit log.
- Mailboxes with end-to-end encryption cannot be connected, because Lockally cannot read them.
- Read-only access never sees the tools that send.

Every message body is returned marked as untrusted third-party content, and messages that look like attempts to manipulate an AI carry an `injection_risk` annotation. Treat email as attacker-controlled input either way.

## Tools

| Tool | What it does | Read-only | Destructive | Open world |
|---|---|---|---|---|
| `list_inboxes` | Lists the mailboxes this connection can reach, with thread counts | yes | no | no |
| `list_threads` | Lists conversations in a mailbox, newest first, or everything since a time | yes | no | no |
| `get_thread` | Returns one whole conversation, in order | yes | no | no |
| `get_message` | Returns one message and its body | yes | no | no |
| `lookup_contact` | Who is this sender: directory record and correspondence history | yes | no | no |
| `analyze_thread` | Intent, urgency, a one-line summary and a suggested next step | yes | no | yes |
| `mark_read` | Marks a message read or unread | no | no | no |
| `reply_or_hold_for_approval` | Saves a reply for a person to approve; sends only when called with `hold: false` and the approval policy allows | no | yes | yes |
| `reply_to_thread` | Sends a reply in an existing conversation | no | yes | yes |
| `send_message` | Starts a new conversation from a mailbox | no | yes | yes |

Reading never marks mail as read. `analyze_thread` sends the subject and short message previews to an AI model provider, needs AI classification switched on by a workspace administrator, and is available from 26 November 2026. See the [privacy policy](https://lockally.com/legal/privacy/), section 6.

`reply_or_hold_for_approval` was called `draft_reply` before Lockally 1.7.0. To keep the old behaviour, call it with `hold: false`.

## Skill

[`skills/triage-inbox`](skills/triage-inbox/SKILL.md) is an agent skill that works through an inbox: it sorts recent conversations into needs a reply, needs a decision, no action and suspicious, saves replies to the obvious ones for approval, never sends, and reports anything that tries to instruct it.

## Example prompts

- "Which inboxes can you see in Lockally?"
- "Go through support@ and tell me what needs a reply, what needs a decision, and what I can ignore."
- "What did this customer actually agree to about the October invoice?"
- "Draft a reply confirming both corrections and save it for my approval."

## Links

- Documentation: [developers.lockally.com/ai/mcp](https://developers.lockally.com/ai/mcp/)
- Official MCP Registry: [`com.lockally/email`](https://registry.modelcontextprotocol.io/v0.1/servers?search=com.lockally%2Femail)
- Support: [docs.lockally.com/support/contact-support](https://docs.lockally.com/support/contact-support/)
- Privacy: [lockally.com/legal/privacy](https://lockally.com/legal/privacy/) · Terms: [lockally.com/legal/terms](https://lockally.com/legal/terms/)
- Security reports: see [SECURITY.md](SECURITY.md)

## License

The files in this repository are MIT licensed. The hosted Lockally service is provided under the [Lockally terms](https://lockally.com/legal/terms/).
