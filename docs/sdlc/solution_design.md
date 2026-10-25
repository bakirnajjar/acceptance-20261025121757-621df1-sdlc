# Solution design — synthetic password-reset prototype

**Status:** proposed architecture and dependency-ordered implementation plan, not an implemented, tested or deployed application. Contract: [`solution_design.json`](solution_design.json), schema version `1.1`, stage `solution_design`. The approved predecessor requirements are `REQ-001`–`REQ-018`; their stage-output responses supplied no standalone source-reference locator, so requirement IDs are predecessor links rather than invented `stage_output` citations. The prior risk register lists prospective risks, not measured defects.

## Authorized evidence and observed boundary

| Key | Exact source reference | Use |
| --- | --- | --- |
| S | `{source_type: 'user_story', source_id: 'user_story', locator: 'characters 0-16000'}` | Python 3.12, Flask, server rendering, pytest, synthetic accounts, in-memory notification sink, limitations, no deployment or deployment workflow. |
| W | `{source_type: 'attachment', source_id: '23b23ac1-a532-42fc-9973-51a1d4954aa6', locator: 'sign-in-wireframe.png'}` | Only the pictured sign-in presentation and visible recovery entry. |
| K | `{source_type: 'knowledge', source_id: 'bcf198d8-a628-401c-8641-b726fcbd00b5', locator: 'v1, chunk 0'}` | ACC-STD-01 identical known/unknown response and five requests/account/hour; ACC-STD-02 one-time links expiring after 30 minutes and every-session sign-out; ACC-STD-03 owner notice after every successful reset. |

**Observed (W):** the 480×360 white canvas has a narrow horizontally centered dark-bordered panel, bold “Sign in” heading, labeled “Email address” field with `name@example.com` placeholder, labeled masked “Password” field, blue underlined “Forgot password?” link, wide dark “Sign in” button and muted “No account? Contact your administrator.” footer. The image establishes neither working sign-in nor recovery-page layouts, authentication rules or actual accessibility behavior. The following interactions and unpictured recovery states are **proposals** based on S, K and approved requirements, not observations from W.

## Architecture and interfaces

The proposed **Python 3.12 Flask app factory** supplies Jinja server-rendered routes and injects a synthetic account store, guarded in-memory state, aware-UTC clock and private in-memory notification sink. `pip` declares Flask and pytest dependencies. Separate app instances have independent state and clocks. There is one process-local state owner, not durable shared storage. A **minimal provisional sign-in/session adapter** verifies synthetic passwords and uses opaque server-side sessions with a guarded demonstration page, exclusively so independent browser clients can prove all-session invalidation. This is not a broader sign-in product contract. [S, W, K]

The **request service** normalizes the proposed email identifier consistently, looks up synthetic accounts, checks a per-existing-account rolling hour of *accepted issuances*, generates a cryptographically unpredictable URL-safe secret for an eligible request, stores its digest (not its plaintext) in grant state and appends the configured-origin reset URL to the private sink. Unknown or over-limit requests do not issue a link. A request's **visitor-facing result never identifies which case occurred**. Under a process-local lock, accepted issue times at or before `now_utc - 1 hour` are pruned; fewer than five remaining times permit a new issue. The sixth request within a window creates no link, whereas eligibility returns at the oldest issue time plus one hour. There is no unsupported global quota or public account-specific `429`. [S, K]

The **completion service** reads a grant's digest, account and aware-UTC issue time. A valid completion needs an unused grant and `now_utc < issued_at_utc + 30 minutes`; equality is expired. It hashes the new nonempty password before taking the lock, then rechecks eligibility inside it and performs a single rollback-safe in-memory transition: consume grant, replace salted password hash, increment account session generation and append one owner-addressed successful-reset notice to the sink. It reports success only after the whole transition commits, including the notice. Concurrent/replayed/expired attempts cannot report success or alter credentials; every old server-side session is rejected on its *next guarded request*, including separate clients. Merely viewing a link does not consume it. Additional revocation of other outstanding grants is optional, not a new acceptance obligation. [S, K]

### Proposed HTTP contracts (not pre-existing endpoints)

| Interface | Request and response/effect |
| --- | --- |
| `GET /sign-in` | Render the pictured sign-in HTML and “Forgot password?” href to `/forgot-password`; no fixture password or sink contents. |
| `POST /sign-in` | Proposed CSRF-checked email/password submission; generic invalid-credential result, or opaque HttpOnly cookie and redirect to a guarded demonstration page. This behavior is provisional. |
| `GET /forgot-password` | Render proposed labeled email-request form and sign-in return path. |
| `POST /forgot-password` | CSRF-checked email submission; equivalent valid-form known/unknown/over-limit requests have the same status, redirect destination, final neutral body and account-independent headers. Malformed form or CSRF errors are input errors, not account-lookup signals. No token, per-account quota or issuance decision in the result. |
| `GET /reset/<token>` | Check current usability without consuming grant; render a proposed new-password form or generic invalid/used/expired guidance and path to request another link. |
| `POST /reset/<token>` | CSRF-checked new nonempty password; recheck grant at submission time, complete at most once, then redirect to a token-free completion URL. Failure never claims success. |
| `GET /reset/complete` | Render success guidance only after a successful transition; direct access without success state gets neutral fallback guidance. |
| Authenticated page guard | Resolve opaque cookie against a server-side session record and the account's current generation on **every** authenticated request. |

Only sign-in has pictured visual styling. For proposed request, neutral confirmation, new-password, unusable-link and completion pages, use server-rendered semantic labels, keyboard-operable controls, visible focus and accessible status/error feedback. No client renderer is necessary. Test labels, contrasts and focus rather than assuming the image establishes accessibility. There is **no public token-inspection or notification-inbox route**: local pytest fixtures inspect the injected sink independently of visitor responses. [S, W, K]

### State and data boundaries

| Prototype record | Data and ownership |
| --- | --- |
| `Account` | Synthetic ID, normalized email, salted password hash, session generation; never real user data. |
| `ResetGrant` | Token digest, account ID, aware-UTC issue time and consumed flag. Link secret is not stored with the grant; the full URL exists only in the private sink record for testing. |
| `RequestLedger` | Account ID → accepted issue timestamps in UTC; guarded per-account sliding-window updates. |
| `SessionRecord` | Random opaque session digest, account ID and issued generation; reject every generation-stale record after reset. A Flask client-side signed cookie alone is insufficient as sole session authority. |
| `Notification` | Kind (`reset_link` or `password_reset_notice`), owner account recipient, UTC time and payload. Success notices have no token or password; unknown accounts cannot create an account-specific link. |

No database or schema migration is in scope. State, rate limits, sessions and sink contents disappear on restart and can diverge across workers; this is **not production-safe persistence, rate limiting, single-use enforcement across workers or actual mail delivery**. Future use of real accounts, a shared atomic store, durable delivery/outbox and any migration plan needs separate authorization; demo identities must not be silently imported. [S, K]

### Security, reliability, observability and decisions

Use salted credential hashing, unguessable link/session secrets, digest-backed grants, protected mutating POSTs, HttpOnly session cookies, a fixed configured reset-link origin instead of a request-controlled `Host`, and no-referrer/no-store controls on token-bearing pages without third-party assets. Do not put link secrets, passwords or emails in application logs or public responses; redact or avoid token-path access logs. The sink is private, not a delivery system; URL-bearing secrets remain sensitive inside it. Account-independent HTTP outcomes do **not** imply constant-time whole-request processing. Emit non-identifying aggregate counters for eligible issuances, quota suppression, invalid/expired/replayed submissions and successful resets; log internal errors with redaction only. Process-local locking and failure-safe updates handle races inside one process, not crashes or multiple workers. [S, K]

| Significant proposed choice | Alternative considered | Reason/tradeoff |
| --- | --- | --- |
| Flask/Jinja and local injected state/sink | SPA/API, database or external mail provider | Meets the required server-rendered synthetic prototype; deliberately non-durable and not deployable. |
| Rolling accepted-issuance ledger | Fixed-hour buckets or public per-account quota response | Exact one-hour boundary and neutral outward result; restarts reset the ledger. |
| Opaque secret with digest-backed grant | Predictable IDs, plaintext grant store or stateless signed token | Supports one-time consumption/revocation and reduces grant-table exposure; URL/sink still hold a secret. |
| Server-side generation-checked opaque sessions | Client-only signed cookies or revoking just current client | All independently active sessions fail after one reset; process-local state remains a prototype limit. |
| Guarded completion and notice append | Asynchronous external send after reporting success | Every reported success has an owner-addressed sink record; a production transactional outbox is outside scope. |
| Configured-origin link and fixture-only sink inspection | Untrusted Host-based URLs or public debug inbox | Avoids link redirection and exposing test links to unauthenticated visitors. |
| Minimal synthetic sign-in | Dead screenshot button or fully specified auth product | Allows `REQ-009` verification without asserting unpictured behavior; broader semantics remain open. |

**Hosting:** `none`; no deployment, hosting configuration, deployment workflow, external delivery, production credentials or rollout is authorized. **Future coding-stage quality gates, from the repository root after the scaffold exists:** build `[]`, test `python3.12 -m pytest -q`, lint `[]`. No app or test suite currently exists and no gate is claimed to have passed. Python 3.12, Flask, Jinja, pytest and pip are selected; a speculative Ruff dependency is not assumed. [S]

## Dependency-ordered implementation tasks

These are proposed coding-stage IDs, not the document-task orchestrator's IDs. Acceptance criteria below are proposed future checks, not completed tests. Task-local sources use the **exact** S/W/K references above; W supports only pictured sign-in/entry details.

### TASK-001 — Python/Flask synthetic-state foundation
**Requirements:** `REQ-011`, `REQ-012`, `REQ-013`, `REQ-014`. **Depends on:** none. **Sources:** S.

Declare Python 3.12 Flask/pytest dependencies; create a Flask app factory with injected UTC clock, isolated synthetic fixtures, guarded process-local account/grant/ledger/session state and private in-memory sink. Acceptance: (1) Flask starts/accepts requests under Python 3.12 and two pytest app factories have isolated accounts, clocks, stores and sinks; (2) fixtures inspect synthetic-only sink records without external delivery or public HTTP inspection; (3) state supports guarded atomic process-local issuance and completion without multi-worker/durability claims.

### TASK-002 — Pictured sign-in and session guard
**Requirements:** `REQ-001`, `REQ-002`, `REQ-009`, `REQ-015`. **Depends on:** `TASK-001`. **Sources:** S, W, K.

Render pictured sign-in and recovery link; provisionally verify synthetic credentials, issue opaque server-side sessions and guard a demonstration page on each request using account generation. Acceptance: (1) HTML presents all pictured panel, heading, labeled inputs/placeholder/masking, blue underlined link, dark wide button and muted exact footer; (2) link reaches server-rendered request entry without client rendering; (3) two clients hold distinct guarded sessions for one account, revocation rejects each, and bad credentials cannot grant a session. This is not a wireframe claim about functional sign-in.

### TASK-003 — Private request and link-issuance service
**Requirements:** `REQ-003`, `REQ-004`, `REQ-005`, `REQ-011`, `REQ-012`. **Depends on:** `TASK-001`. **Sources:** S, K.

Normalize email; guard per-account rolling accepted issuance times; eligible known requests create digest-backed opaque grants and fixed-origin links solely in the private sink. Acceptance: (1) five links inside an hour, no sixth, eligibility at oldest issue plus one hour; (2) unknown identifiers have no account-specific link and separate accounts have independent ledgers; (3) state retains token digest, account ID and UTC issue time but not plaintext, while caller-facing outcome hides existence, quota and link.

### TASK-004 — Request form and neutral HTTP result
**Requirements:** `REQ-002`, `REQ-003`, `REQ-004`, `REQ-015`. **Depends on:** `TASK-002`, `TASK-003`. **Sources:** S, W, K (W only for entry).

Add proposed Flask/Jinja email form and confirmation. Acceptance: (1) pictured link reaches a server-rendered protected POST form; (2) equivalent valid known/unknown/over-limit submissions have identical visitor-facing status, redirect, final body and account-independent headers without sink/token disclosure; (3) neutral confirmation renders server-side, and form/CSRF errors do not expose account status.

### TASK-005 — One-time reset transaction and all-session revocation
**Requirements:** `REQ-006`, `REQ-007`, `REQ-008`, `REQ-009`, `REQ-010`, `REQ-012`. **Depends on:** `TASK-002`, `TASK-003`. **Sources:** S, K.

Hash submitted replacement; under guarded state recheck digest and time, consume once, replace password hash, increment session generation and append exactly one owner notice before success. Acceptance: (1) eligible strictly before 30 minutes, never twice even on concurrent replay; (2) at/after the exact boundary no credential, generation or notice change; (3) new password accepted and old rejected, **all** prior clients unauthenticated; (4) one owner-addressed notice per reported success, no token/password therein and none for invalid/replayed/expired submissions.

### TASK-006 — Server-rendered link, error and completion states
**Requirements:** `REQ-005`, `REQ-006`, `REQ-007`, `REQ-008`, `REQ-015`. **Depends on:** `TASK-004`, `TASK-005`. **Sources:** S, K.

Add proposed valid-link form, generic invalid/used/expired guidance, protected POST and token-free, success-gated completion; use configured-origin links and anti-leak headers. Acceptance: (1) valid GET renders labeled form without consumption; unusable links lead to a new request; (2) protected valid POST completes once and redirects to token-free URL, invalid/expired/replayed POST never mutates or claims success; (3) direct completion access does not falsely assert success and pages leak no account, sink or password, while token-bearing pages suppress cache/referrer leakage.

### TASK-007 — Automated entry and request-flow verification
**Requirements:** `REQ-001`, `REQ-002`, `REQ-003`, `REQ-004`, `REQ-005`, `REQ-011`, `REQ-012`, `REQ-013`, `REQ-014`, `REQ-015`, `REQ-016`. **Depends on:** `TASK-006`. **Sources:** S, W, K.

Add Python 3.12 pytest/Flask-client fixtures and request-flow tests. Acceptance: (1) pictured server-rendered sign-in and request navigation without JavaScript; (2) compare equivalent known/unknown/over-limit HTTP status, redirect, body and account-independent headers separately from eligible-link/unknown-no-link sink assertions; (3) injected clock checks five links, blocked sixth, independent quotas and elapsed-hour boundary; (4) fresh synthetic app fixtures have no prior accounts, quota or notices.

### TASK-008 — Automated reset and session verification
**Requirements:** `REQ-006`, `REQ-007`, `REQ-008`, `REQ-009`, `REQ-010`, `REQ-012`, `REQ-015`, `REQ-016`. **Depends on:** `TASK-006`. **Sources:** S, K.

Add independent pytest reset lifecycle/guard tests with UTC clock and at least two authenticated clients. Acceptance: (1) pre/equal/post 30-minute boundary, non-consuming GET and POST recheck; (2) one completion and rejection of sequential/concurrent replay without second credential change or notice, old password rejected and new accepted; (3) every pre-existing client loses authentication, each success has exactly one owner notice and failures none; (4) unusable-link and token-free completion HTML is server-rendered without public link/sink disclosure. With TASK-007 this tests every mandated `REQ-016` scenario.

### TASK-009 — Document limitations and verify non-deployment scope
**Requirements:** `REQ-017`, `REQ-018`. **Depends on:** `TASK-007`, `TASK-008`. **Sources:** S.

Document prototype limitations and conduct a **read-only** deployment-scope check, not a rollout. Acceptance: (1) project documentation names synthetic accounts, process-local state, sink-only messages, restart/multi-worker loss and absent real delivery as production limitations; (2) inspect delivered changes and workflow files for no deployment workflow and perform no deployment; (3) record inspection evidence without implying production infrastructure was built.

## Requirement coverage and dependency check

| Requirement | Implementing/verifying tasks | Requirement | Implementing/verifying tasks |
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

All 18 approved requirements map to tasks: **coverage dispositions `[]`**. `REQ-018` is a scope/documentation check, **not** a deployment task. All edges point to earlier IDs: `TASK-001` → `TASK-002`/`TASK-003` → `TASK-004`/`TASK-005` → `TASK-006` → `TASK-007`/`TASK-008` → `TASK-009`; `TASK-005` also requires `TASK-002` and `TASK-003`.

## Assumptions and open questions

Assumptions: email is the synthetic account identifier; only the sign-in visual is observed; a process-local injected clock, private sink and state owner permit deterministic tests; a provisional sign-in/session guard exists solely to test `REQ-009`; no additional password rules, real accounts, production persistence or recovery-page visuals are evidenced. Prior stage identifiers are links, not source citations. None of these is a claim of implemented behavior.

All questions are **nonblocking**: What should the Sign in button do beyond a synthetic-session demonstration? Is specific unpictured recovery-page layout/copy expected? Should future policy prescribe a stronger password rule or different session mechanism? Should a separately authorized production stage define durable accounts/shared quotas/external owner delivery and secure local inspection? No answer expands this scope or permits deployment.
