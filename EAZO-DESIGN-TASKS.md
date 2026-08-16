# EAZO → Zhizi design-task protocol (live canary)

EAZO may submit one narrow, public-safe design review to Zhizi without asking
Macheng to copy/paste the result between systems.

## Current live allowlist

- `need` must be exactly `design_review`.
- The only accepted repository is
  `https://github.com/MachengShen/starshard-layered-feedback`.
- The request may ask only for review, critique, prioritization, wireflow, or
  testable acceptance criteria. Submission is a request, not authorization.
- No secrets, private transcripts/source, identity/payment data, external
  sends, account changes, deletion, deployment, or private Owner Plane access.
- `design_handoff`, `code_patch_candidate`, and other repositories are reserved
  for later rollout and are currently owner-gated.

## Submit

POST JSON to:

`https://eazo.clawishmacheng.com/api/eazo/agent-turn`

```json
{
  "intent": "Review the public Starshard layered-feedback design and return a prioritized design handoff.",
  "message": "Preserve strong ideas, identify P0/P1/P2 issues, and give testable mobile acceptance criteria. Do not write code or perform external actions.",
  "need": "design_review",
  "modality": "text",
  "declared_name": "EAZO Design Agent",
  "conversation_id": "stable-eazo-conversation-id",
  "trace_id": "unique-eazo-turn-id",
  "client_receipt_id": "stable-eazo-task-id",
  "context": "Public-safe design review only.",
  "imageUrl": "https://github.com/MachengShen/starshard-layered-feedback",
  "artifact_urls": [
    "https://github.com/MachengShen/starshard-layered-feedback"
  ],
  "agent_state": {
    "task_type": "design_review",
    "permission_request": "bounded_public_repo_work_only"
  },
  "client_metadata": {
    "source_view": "eazo-design"
  }
}
```

Store the returned `receipt_id`. Do not treat it as a credential or an approval
token.

## Poll and close

Poll:

`https://eazo.clawishmacheng.com/api/eazo/status?receipt_id={receipt_id}`

Start at 60-second intervals, then back off after five minutes. Display the
returned status. Treat only `completed` plus a public GitHub result pointer as
success. Surface `owner_gate_required`, `timed_out`, or
`blocked_with_repair_plan` as non-success states; never retry them with broader
authority.

The initial intake checks at five-minute intervals and processes at most one
new receipt per run. This is intentionally a narrow canary, not a general tool
execution API.
