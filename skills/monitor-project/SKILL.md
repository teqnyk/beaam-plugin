---
name: monitor-project
description: >
  This skill should be used when the user asks to "monitor this project with
  Beaam", "audit this repository's monitoring", "set up monitoring for this
  app", "find monitoring gaps", or wants Beaam coverage derived from a local
  codebase, deployment configuration, dependencies, or public URLs.
metadata:
  version: "1.0.0"
  author: "Teqnyk Ltd"
---

# Monitor a project with Beaam

Build a truthful, consent-based monitoring plan from local project evidence,
apply only what the user approves, and verify the resulting coverage.

## Principles

- Start with a read-only audit. Do not connect providers, create monitors, or
  change Beaam until the user has seen and approved the stored plan.
- Fetch Beaam's current integration catalog before scanning. Treat its
  detection fingerprints and gathering guidance as authoritative; do not rely
  on a remembered provider list.
- Send evidence, never secrets. Read environment-variable names, not values.
- Never turn missing evidence, a failed provider call, or an incomplete
  inventory into a plausible-looking success. Name every gap and failure.
- Treat an installed dependency as weak evidence. Prefer deployment files,
  authenticated identity checks, matching environment-variable names, or an
  explicit user answer before concluding that a provider is deployed.
- Keep the approved plan and applied plan identical. Use the returned plan ID
  and exact stored option values; never reconstruct or widen the actions.

## 1. Establish scope

Confirm which repository or workspace is in scope when it is ambiguous. For a
monorepo, inspect every relevant application rather than only the root package.
Ask which Beaam stack should receive the monitors only when Beaam returns a
stack-selection question; do not invent a destination.

Do not require the user to choose integrations up front. The audit exists to
discover them.

## 2. Fetch the live catalog

Call `get_integration_catalog` before reading the project. Follow the returned
`guidance.read`, `guidance.neverRead`, and `guidance.notes` rules exactly.

Use only entries whose current status and fingerprints support the evidence.
Report planned or withdrawn integrations as unavailable rather than treating
them as connectable.

## 3. Gather safe local evidence

Collect a bounded inventory for `plan_monitoring`:

1. Record normalized, repository-relative file paths, preserving nested paths.
2. Read every relevant `package.json` and gather dependency and dev-dependency
   names.
3. Inspect example environment files such as `.env.example` and `.env.sample`,
   including commented declarations, but record variable names only.
4. Find environment-variable names referenced by source code, such as
   `process.env.NAME`, `env.NAME`, and `import.meta.env.NAME`.
5. Inspect deployment configuration at the repository root and one level down.
6. Record API hostnames statically referenced by source code. Do not send paths,
   query strings, headers, request bodies, or credentials.
7. Gather candidate public production URLs from deployment configuration,
   public metadata, or the user's explicit answer. Exclude localhost, private
   addresses, preview URLs presented as production, and anything secret-bearing.
8. Record non-secret provider account or organization identifiers only when
   they are already present in deployment configuration or returned by an
   approved identity check.

Never open `.env`, `.env.local`, `.env.production`, credential stores, shell
history, cloud credential files, `.netrc`, browser storage, or files containing
live secret values. Never include environment-variable values in evidence or in
the response.

If the catalog publishes an identity command, run it only when that executable
is already installed. Run the exact command without modification, never install
or authenticate a CLI, and reduce the result to `{ command, authenticated }`.
Discard stdout and stderr; do not quote, summarize, log, or send them to Beaam.

## 4. Generate and explain the plan

Call `list_stacks` when stack context is needed, then call `plan_monitoring` once
with the complete safe evidence set and, when explicitly selected, `stackId`.

Present the result under these headings:

- Already covered
- Proposed connections
- Public URLs to watch, including reachability results
- Decisions needed
- Cannot cover
- Plan limits

For each proposed provider, show the human-readable evidence, confidence, setup
method, and target stack. Clearly distinguish strong deployment evidence from a
weak dependency-only match. Surface every item in `cannot` and every failed or
unreachable URL.

Ask each returned question using its wording and options. Preserve the option
values internally and display the labels to the user. Never invent an answer,
substitute a similar value, or silently select the first option.

The plan is single-use and expires after 30 minutes. If it expires or the user
changes scope, gather current evidence and generate a new plan rather than
trying to reuse it.

## 5. Obtain approval

Before calling `apply_monitoring_plan`, summarize exactly what it will change:
the URLs that will be watched, providers that will begin connection, target
stack, and any unresolved gaps. Ask for explicit approval.

Treat approval as applying only to that displayed plan. A request for changes
requires a new plan or new answers, not an edited action list.

## 6. Apply without widening

Call `apply_monitoring_plan` with the returned `planId` and answers copied
exactly from the plan's stored option values.

Report its result faithfully:

- `connected`: completed automatically.
- `failed`: attempted but failed; include the provider's actionable reason.
- `cannot`: still outside Beaam's coverage.
- `next` / `needsBrowser`: user-authorized OAuth steps.
- `needsToken`: a secret must be supplied by the user through Beaam's secure
  connection flow, never through the conversation or local project scan.

For OAuth, handle one provider at a time. Give the user `next.url`, poll
`check_oauth_connect` with the returned state after authorization, then call
`next_connection` for the next provider. Do not open multiple authorization
flows concurrently or claim success before the completion check passes.

Never ask the user to paste an API token, password, private key, or OAuth code
into the conversation. Direct token-based setup to Beaam's secure web flow.

## 7. Verify coverage

After all feasible steps finish:

1. Re-read the relevant integrations, services, stacks, and status board.
2. Confirm each automatically added URL or completed provider appears in the
   intended stack.
3. Distinguish a newly connected service awaiting its first check from a
   healthy service. Do not call a pending or unknown state healthy.
4. Report every remaining OAuth step, token step, tier limit, unsupported
   provider, unreachable target, collection failure, or unverified assumption.

Finish with a compact coverage report containing completed coverage, pending
user actions, unsupported gaps, and the next most valuable step. If nothing was
changed, say so explicitly.
