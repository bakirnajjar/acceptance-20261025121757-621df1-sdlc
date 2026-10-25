# Solution-design implementation task map (T002)

**Status:** approved planning artifact for the later solution-design contract, not an implemented application or final `docs/sdlc` output. `TASK-001`–`TASK-009` are prospective coding-stage identifiers, **not** orchestrator keys `T001`/`T002`. Preserve these task IDs and predecessor requirement IDs when constructing the final contract. All dependencies point backward; none call for deployment.

## Evidence and task-local citation convention

The following are the **only** source references used below, with the returned IDs and locators reproduced exactly. For each task, its “Sources” line names the applicable references from this list; carry the corresponding full objects into that task's `source_references` array in the final contract.

| Shorthand | Full source reference | Scope |
| --- | --- | --- |
| S | `{source_type: 'user_story', source_id: 'user_story', locator: 'characters 0-16000'}` | Python 3.12/Flask, server rendering, recovery prototype, pytest, synthetic accounts, in-memory sink, limitations, no deployment. |
| W | `{source_type: 'attachment', source_id: '23b23ac1-a532-42fc-9973-51a1d4954aa6', locator: 'sign-in-wireframe.png'}` | Pictured sign-in presentation and visible recovery affordance **only**. |
| K | `{source_type: 'knowledge', source_id: 'bcf198d8-a628-401c-8641-b726fcbd00b5', locator: 'v1, chunk 0'}` | ACC-STD-01 neutral existing/unknown response and five/account/hour; ACC-STD-02 single use, 30-minute expiry and all-session sign-out; ACC-STD-03 owner notice on every successful reset. |

Approved `requirements_analysis`, `ui_analysis` and `acceptance_20261025121757_621df1_risk` were read through authorized stage outputs but had **no returned standalone source-reference locator**. Link their approved `REQ-001`–`REQ-018` identifiers without inventing a `stage_output` reference. Risk entries are prospective, not discovered defects. Recovery-page layouts, authentication details, password rules and production infrastructure are not pictured; specifics below are **proposed implementation choices**, not new requirements.

## Dependency-ordered, independently reviewable tasks

### TASK-001 — Python/Flask synthetic-state foundation

- **Requirements:** `REQ-011`, `REQ-012`, `REQ-013`, `REQ-014`
- **Depends on:** none.
- **Sources:** S.
- **Description:** Declare Python 3.12 and Flask/pytest dependencies and build a Flask app factory with injected aware-UTC clock, isolated synthetic account fixtures, a single-process guarded state owner and an in-memory notification sink. Define minimal account, grant, issuance-ledger, session and notification records/interfaces for dependent tasks; keep real accounts, databases and external delivery out of scope. App-factory injection makes state and clock separately inspectable by tests without adding public sink routes.
- **Acceptance criteria:**
  1. Under Python 3.12 with declared dependencies installed, the Flask application starts and accepts a request; a pytest fixture can create two isolated app instances with separate accounts, clocks, stores and sinks.
  2. Accounts used by the recovery flow are explicitly synthetic; notifications can be inspected through the injected in-memory sink in tests without a real delivery service or HTTP inspection endpoint.
  3. The state interface supports atomic, single-process guarded updates for issuance and completion; it makes no durability or multi-worker claim.

### TASK-002 — Pictured sign-in and session guard

- **Requirements:** `REQ-001`, `REQ-002`, `REQ-009`, `REQ-015`
- **Depends on:** `TASK-001`.
- **Sources:** S, W, K.
- **Description:** Server-render the pictured sign-in page with a working recovery entry link. Add only a **provisional** synthetic-account password-verifying sign-in, opaque server-side session records and an authenticated demonstration guard so distinct test clients can establish independently active sessions; check account session generation on every guarded request. Sign-in behavior beyond the pictured controls is a design mechanism for testing `REQ-009`, not a wireframe claim or expanded product requirement. Use a salted password hash and protected form mutations; do not expose fixture secrets on the page.
- **Acceptance criteria:**
  1. The HTML response presents a narrow dark-bordered sign-in panel on white, a “Sign in” heading, labeled “Email address” with `name@example.com` placeholder, labeled masked “Password” input, blue underlined “Forgot password?” link, wide dark “Sign in” button, and muted “No account? Contact your administrator.” footer.
  2. Following the recovery link reaches the server-rendered request entry; the pictured page does not require client-side rendering.
  3. Two independent browser clients can acquire distinct guarded sessions for the same synthetic account; each guarded request checks server-side validity/generation and rejects a revoked session. Invalid sign-in credentials do not grant a session.

### TASK-003 — Private request and link-issuance service

- **Requirements:** `REQ-003`, `REQ-004`, `REQ-005`, `REQ-011`, `REQ-012`
- **Depends on:** `TASK-001`.
- **Sources:** S, K.
- **Description:** Normalize email consistently for synthetic account lookup and implement a guarded rolling-hour **accepted issuance** ledger keyed by account. For an eligible existing account, generate an unpredictable opaque secret, retain only its digest in grant state, and record its fixed-origin reset URL solely in the private in-memory sink. Unknown and over-limit requests create no grant or link; return an account-independent service outcome for the presentation task. Prune accepted times at or before `now - 1 hour`; the five-per-account window does not become a public quota signal.
- **Acceptance criteria:**
  1. Five eligible requests for one account inside a rolling hour record five account-addressed links; a sixth inside that window records no sixth link; at the oldest issue time plus one hour a further eligible request records a link.
  2. A request for an unknown email records no account-specific link; requests for other synthetic accounts use independent ledgers.
  3. Grant state stores a token digest, account ID and UTC issuance time, not the plaintext secret; only the injected sink holds the test-inspectable URL, and caller-facing service outcomes do not identify existence, quota or link contents.

### TASK-004 — Request form and neutral HTTP result

- **Requirements:** `REQ-002`, `REQ-003`, `REQ-004`, `REQ-015`
- **Depends on:** `TASK-002`, `TASK-003`.
- **Sources:** S, W, K (W supports the entry affordance, **not** recovery-screen appearance).
- **Description:** Add Flask/Jinja request and neutral-confirmation pages with a proposed labeled email form and return route. Route valid POSTs through `TASK-003`; equivalent known, unknown and over-limit identifiers produce the same visitor-facing result. Validate form input and CSRF separately from account lookup; do not render sink contents, a token or per-account quota detail. Recovery-page layout/copy and accessible feedback are proposed design choices, not pictured requirements.
- **Acceptance criteria:**
  1. The pictured link opens a server-rendered page where a visitor can submit an email identifier through a protected POST.
  2. Equivalent valid-form requests for known and unknown identifiers, and over-limit known requests, have identical visitor-facing status, redirect target, final response body and account-independent headers; no response or page reveals an account, issuance decision, token or sink record.
  3. A neutral confirmation is rendered without client-side rendering; invalid-form/CSRF handling does not imply an account-existence result.

### TASK-005 — One-time reset transaction and all-session revocation

- **Requirements:** `REQ-006`, `REQ-007`, `REQ-008`, `REQ-009`, `REQ-010`, `REQ-012`
- **Depends on:** `TASK-002`, `TASK-003`.
- **Sources:** S, K.
- **Description:** Validate grant digest and injected UTC time on each completion. Within one guarded in-memory state transition, consume a valid unused grant, replace the synthetic account's password hash, advance its session generation to invalidate **every** existing session, and append one owner-addressed successful-reset notice to the sink before reporting success. Hash before taking the state lock, then recheck eligibility inside the lock; no reported success if any committed effect fails. Token validity is strict `now < issued_at + 30 minutes`; GET/inspection must not consume a grant. Revoking other outstanding account grants is an optional conservative implementation decision, not a separate acceptance requirement.
- **Acceptance criteria:**
  1. A valid unused link submitted strictly before `t + 30 minutes` can replace the password exactly once; a repeat or concurrent replay cannot complete a second reset.
  2. At exactly `t + 30 minutes` and afterward, an attempt fails without changing the password, session generation or notification count.
  3. After success, old password verification fails and new password verification succeeds; **all** previously authenticated clients for that account fail the guard, not just the submitting client.
  4. Each reported successful reset records exactly one owner-addressed notice in the in-memory sink; invalid/replayed/expired attempts record no success notice. Secrets and new passwords are absent from success notices.

### TASK-006 — Server-rendered link, error and completion states

- **Requirements:** `REQ-005`, `REQ-006`, `REQ-007`, `REQ-008`, `REQ-015`
- **Depends on:** `TASK-004`, `TASK-005`.
- **Sources:** S, K.
- **Description:** Add server-rendered valid-link new-password form, invalid/used/expired-link guidance with a route to request another link, and token-free completion guidance. Only a protected POST invokes the reset service; GET checks usability without consumption. Revalidate on POST even when an earlier GET rendered a form. Use a fixed configured origin when generating links; keep tokens and sink contents off public results and avoid referrer/cache leakage from token-bearing pages. These recovery states are proposed, not additional wireframe observations or a prescribed password-strength policy.
- **Acceptance criteria:**
  1. A valid unused link opens a fully server-rendered labeled new-password form without consuming the grant; an invalid, expired or used link opens a generic unusable-link state with a recovery-request path.
  2. A valid protected POST can complete exactly one reset and redirect to a token-free completion URL; later, invalid and expired POSTs cannot report success or mutate credentials.
  3. Direct access to completion without success state does not falsely assert that a reset succeeded; pages reveal no account-specific grant, sink contents or password, and token-bearing pages suppress caching/referrer leakage.

### TASK-007 — Automated entry and request-flow verification

- **Requirements:** `REQ-001`, `REQ-002`, `REQ-003`, `REQ-004`, `REQ-005`, `REQ-011`, `REQ-012`, `REQ-013`, `REQ-014`, `REQ-015`, `REQ-016`
- **Depends on:** `TASK-006`.
- **Sources:** S, W, K.
- **Description:** Add Python 3.12 pytest fixtures and Flask-client tests for the pictured HTML/navigation, synthetic/in-memory boundaries, neutral HTTP behavior, private issuance and rolling-hour boundary. Inspect sink changes independently from visitor response equivalence; use injected time and isolated app factories, not sleeps or public sink pages.
- **Acceptance criteria:**
  1. Pytest under Python 3.12 verifies Flask-served, server-rendered sign-in controls/presentation and navigation to a server-rendered reset-request form without JavaScript.
  2. Tests compare status, redirect, resulting body and account-independent headers for equivalent known/unknown/over-limit valid submissions, while separately asserting the private sink has eligible-account links but no unknown-account link.
  3. Tests verify five accepted issuances, no sixth inside one rolling hour, independent account quotas, and renewed eligibility at the oldest accepted issue time plus one hour, using an injected clock.
  4. Tests run against synthetic fixtures and in-memory sink only; a fresh app fixture has no prior account, notification or quota state.

### TASK-008 — Automated reset and session verification

- **Requirements:** `REQ-006`, `REQ-007`, `REQ-008`, `REQ-009`, `REQ-010`, `REQ-012`, `REQ-015`, `REQ-016`
- **Depends on:** `TASK-006`.
- **Sources:** S, K.
- **Description:** Add independent pytest/Flask-client tests for reset-link lifecycle, atomic replay protection, credential replacement, cross-client sign-out, owner-addressed notices and server-rendered error/success states. Use injected UTC clock, private sink fixtures and at least two distinct authenticated clients; explicitly test the mandated `REQ-016` reset scenarios.
- **Acceptance criteria:**
  1. Tests exercise valid submission immediately before the 30-minute boundary and rejection exactly at and after it; a valid-link GET does not consume the grant and a later POST rechecks validity.
  2. Tests prove one successful completion per link, rejection of sequential and concurrent replay without another credential change or notice, and old password rejected/new password accepted.
  3. Tests authenticate multiple independent clients for one account and verify **each** old session is unauthenticated after successful reset; each success has one owner-addressed sink notice, while unsuccessful submissions have none.
  4. Tests verify invalid/used/expired pages and token-free completion are server-rendered, with no sink inspection route or client-visible link disclosure. Together with `TASK-007`, pytest covers **all** mandated `REQ-016` scenarios: pictured entry, neutral response, per-account limit, issuance, single use, expiry, password replacement, all-session sign-out and owner notice.

### TASK-009 — Document limitations and verify non-deployment scope

- **Requirements:** `REQ-017`, `REQ-018`
- **Depends on:** `TASK-007`, `TASK-008`.
- **Sources:** S.
- **Description:** Write prototype limitations for project readers and conduct a read-only change/scope check. Explain synthetic identities, ephemeral single-process accounts/grants/quotas/sessions, in-memory-only notifications, no external delivery or durable/multi-worker enforcement, and no production-readiness claim. This is a documentation/verification slice **only** for `REQ-018`: do not deploy, add deployment workflow files, write deployment instructions as if a rollout were planned, or treat an absence-of-deployment claim as proven by a runtime test.
- **Acceptance criteria:**
  1. Project documentation explicitly identifies synthetic accounts and the in-memory sink as unsuitable production substitutes and explains restart/multi-worker state loss and absent real delivery.
  2. Review of delivered changes and workflow files finds no deployment workflow added; delivery involved no deployment action.
  3. The recorded scope-check result is evidence of inspection, not a claim that production infrastructure was built or deployed.

## Requirement coverage matrix

Every approved requirement has at least one implementing/verifying task. **No “not implemented” coverage disposition is proposed.** `REQ-018` is satisfied by a documentation/scope check, not deployment.

| Requirement | Task IDs | Requirement | Task IDs |
| --- | --- | --- | --- |
| `REQ-001` | `TASK-002`, `TASK-007` | `REQ-010` | `TASK-005`, `TASK-008` |
| `REQ-002` | `TASK-002`, `TASK-004`, `TASK-007` | `REQ-011` | `TASK-001`, `TASK-003`, `TASK-007` |
| `REQ-003` | `TASK-003`, `TASK-004`, `TASK-007` | `REQ-012` | `TASK-001`, `TASK-003`, `TASK-005`, `TASK-007`, `TASK-008` |
| `REQ-004` | `TASK-003`, `TASK-004`, `TASK-007` | `REQ-013` | `TASK-001`, `TASK-007` |
| `REQ-005` | `TASK-003`, `TASK-006`, `TASK-007` | `REQ-014` | `TASK-001`, `TASK-007` |
| `REQ-006` | `TASK-005`, `TASK-006`, `TASK-008` | `REQ-015` | `TASK-002`, `TASK-004`, `TASK-006`, `TASK-007`, `TASK-008` |
| `REQ-007` | `TASK-005`, `TASK-006`, `TASK-008` | `REQ-016` | `TASK-007`, `TASK-008` |
| `REQ-008` | `TASK-005`, `TASK-006`, `TASK-008` | `REQ-017` | `TASK-009` |
| `REQ-009` | `TASK-002`, `TASK-005`, `TASK-008` | `REQ-018` | `TASK-009` |

**DAG check:** `TASK-001` → `{TASK-002, TASK-003}` → `TASK-004` and `TASK-005` → `TASK-006` → `{TASK-007, TASK-008}` → `TASK-009` (with `TASK-005` also requiring `TASK-002` and `TASK-003`). No self-dependency, missing predecessor or forward dependency. These are proposed coding-stage tasks; this document does not claim any application, test or deployment has been run.
