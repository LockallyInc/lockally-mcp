# Changelog

## Unreleased

- Seven new tools (held until released): `search_threads`, `list_drafts`, `get_draft`, `get_attachment`, `get_delivery_status`, `hold_new_message`, `cancel_draft`.
- This repository: connection guides, the official MCP Registry entry (`server.json`), the `triage-inbox` skill and examples.
- `server.json` 1.0.3 points the registry entry's repository at this repository. Not yet published to the registry.

## Lockally 1.7.0 (server)

- `draft_reply` is renamed `reply_or_hold_for_approval` and holds replies for approval unless called with `hold: false`.
- Every tool states `readOnlyHint`, `destructiveHint` and `openWorldHint` explicitly.
- A connection signed in with OAuth reaches only the signer's own mailbox and the shared mailboxes they belong to, with an optional two-step code at sign-in.
