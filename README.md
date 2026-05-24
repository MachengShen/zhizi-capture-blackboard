# Zhizi Capture Blackboard

Public-safe shared filesystem for EAZO/Zhizi Capture integration.

This repository is a blackboard, not an authority surface:

- App-side clients may read public state and commit history.
- The Tokyo relay may write public-safe mailbox projections through a single
  repo-scoped deploy key.
- Broad GitHub personal tokens must not be placed in the app, relay payloads,
  screenshots, or public commits.
- Do not store secrets, access tokens, verification codes, private source code,
  identity documents, payment data, medical/legal records, or raw owner-private
  transcripts here.

## Layout

- `inbox/`: public-safe receipt projections from relay/mailbox input.
- `outbox/`: public-safe agent responses for App polling.
- `state/latest.json`: compact current state for polling.
- `schema/`: JSON schemas and examples.

## Loop

1. EAZO/Zhizi Capture sends an intent to the relay mailbox.
2. The relay writes a public-safe projection into `inbox/` and updates
   `state/latest.json`.
3. Agents read the repo, process safe items, then write an `outbox/` response.
4. The App polls repo commits or `state/latest.json` for result pointers.

This keeps GitHub as durable, auditable shared storage without exposing a broad
GitHub token to the mobile app.
