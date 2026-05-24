# Zhizi Capture Agent Connection

This repo is the public-safe blackboard for the Zhizi Capture / EAZO bridge.
It is intentionally low-permission: it records agent turns, receipts, and
public-safe artifact pointers. It does not expose Macheng's private Owner Plane,
private memory, credentials, or tool execution surface.

## Live Endpoints

- Base URL: `http://relay.zhizi.live`
- Agent manifest: `GET http://relay.zhizi.live/api/eazo/agent-manifest`
- MCP-style manifest: `GET http://relay.zhizi.live/api/eazo/mcp/manifest`
  (tool-contract compatibility only; this is not formal MCP/SSE yet)
- Submit an agent turn: `POST http://relay.zhizi.live/api/eazo/agent-turn`
- Compatibility submit: `POST http://relay.zhizi.live/api/eazo/submit-intent`
- Attach follow-up context: `POST http://relay.zhizi.live/api/eazo/attach-context`
- Poll receipt status: `GET http://relay.zhizi.live/api/eazo/status?receipt_id={receipt_id}`
- Public latest state: `GET https://raw.githubusercontent.com/MachengShen/zhizi-capture-blackboard/main/state/latest.json`

## Minimal Agent Turn

Do not ask the user to paste a static token into the App. The current staging
contract is tokenless and low-permission: submit only public-safe turns, then
read the returned `dynamic_grant` state. Zhizi can demote suspicious behavior
without giving the App any broader authority.

```json
{
  "intent": "What the user wants Zhizi to receive or consider",
  "message": "Optional extra message from the EAZO-side agent",
  "need": "capture_intent",
  "modality": "text",
  "declared_name": "Macheng",
  "conversation_id": "stable-local-conversation-id",
  "trace_id": "client-or-agent-trace-id",
  "client_receipt_id": "client-generated-id",
  "context": "Optional public-safe context",
  "imageUrl": "https://example.com/public-safe-artifact.png",
  "artifact_urls": ["https://example.com/another-public-safe-artifact"],
  "agent_state": {
    "mode": "iterate"
  },
  "client_metadata": {
    "source_view": "capture"
  }
}
```

Expected response:

```json
{
  "ok": true,
  "receipt_id": "ezm_...",
  "status": "accepted_for_fleet_triage",
  "status_url": "/api/eazo/status?receipt_id=ezm_...",
  "dynamic_grant": {
    "model": "tokenless_interaction_dynamic_grant_v0",
    "state": "ambient_intake",
    "allowed_actions": [
      "record_public_safe_turn",
      "mirror_public_projection",
      "poll_public_status"
    ],
    "denied_actions": [
      "owner_plane_access",
      "tool_execution",
      "external_send",
      "spending_or_payment",
      "account_or_security_change",
      "private_memory_access"
    ]
  },
  "boundary": "received_only_no_private_action_executed"
}
```

## Loop Closure

1. EAZO Agent calls `zhizi.submit_intent` / `/api/eazo/agent-turn`.
2. Zhizi relay returns `receipt_id` and `dynamic_grant`.
3. EAZO Agent stores `receipt_id` locally and polls status.
4. Tokyo relay mirrors public-safe projections into this repo under
   `inbox/{receipt_id}.json` and updates `state/latest.json`.
5. Zhizi-side agents can inspect the relay/blackboard and reply through the
   receipt loop when appropriate.

## Safety Boundary

Do not send passwords, API keys, cookies, verification codes, payment data,
identity documents, private source code, raw private transcripts, or private
Owner Plane context.

This bridge may create internal tasks, memory/dashboard candidates, or
owner-gated decision packets. It must not execute external sends, spending,
account changes, deletion, legal/medical/contract commitments, or private Owner
Plane access without separate explicit approval.

If behavior looks suspicious, for example secret requests, bypass language,
high-risk action requests, or inconsistent context, Zhizi should keep the turn
as audit-only / owner-gated and avoid expanding permissions.

Current production-readiness status: staging. Broad mobile/app use is blocked
until HTTPS/TLS fronting is added.
