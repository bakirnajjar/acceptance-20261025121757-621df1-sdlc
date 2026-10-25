# Proposed password-reset test-case inventory

**Status:** test design only; no application, tests, delivery, or deployment were executed or inspected by this draft. The 23 acceptance criteria in approved REQ-001–REQ-018 are traced below to approved TASK-001–TASK-009. Criterion numbers are 1-based ordinals within each requirement, not new requirement identifiers. Each numbered step is an action for a future implementation/test; each corresponding expected result is an assertion, not a claim of current behavior.

## Evidence and interpretation

| Key | Exact source reference | Limits |
| --- | --- | --- |
| S | `{source_type: 'user_story', source_id: 'user_story', locator: 'characters 0-16000'}` | Python 3.12, Flask, server-rendered pages, pytest, synthetic accounts, in-memory sink, documentation and no deployment. |
| W | `{source_type: 'attachment', source_id: '23b23ac1-a532-42fc-9973-51a1d4954aa6', locator: 'sign-in-wireframe.png'}` | Observed 480×360 white image and sign-in appearance only; no working navigation, live masking, recovery screen or authentication is observed. |
| K | `{source_type: 'knowledge', source_id: 'bcf198d8-a628-401c-8641-b726fcbd00b5', locator: 'v1, chunk 0'}` | Curated standard: link from sign-in, identical known/unknown reset responses, five requests per account per hour, 30-minute single-use links, all-session sign-out and post-success owner notification. It does **not** mandate CSRF implementation, Flask, sink, token storage, authentication details, page layout, password policy, or delivery provider. |

The approved `requirements_analysis`, `ui_analysis` and `solution_design` supply the REQ/TASK links and proposed HTTP/service contracts. They supplied **no standalone `stage_output` source-reference locator**; no such citation is fabricated here. In particular, `/sign-in`, `/forgot-password`, `/reset/<token>` and `/reset/complete`, guarded synthetic sessions, CSRF checks, atomic process-local state and token-page privacy controls are **approved solution-design proposals**, not observations from W or independent mandates of K. Use a fixture-only private sink, injected aware-UTC clock, isolated app factories, separate Flask clients, and synthetic identifiers such as `alex@example.test`, `bea@example.test` and `missing@example.test`; never use real addresses or external delivery. `old-Secret-1`, `new-Secret-2` and `other-Secret-3` below are distinct **test values**, not a prescribed password policy. Avoid sleeping, public link-inspection routes and assuming client-side rendering.

## Cases

### TC-001 — Pictured sign-in content and layout
**Links:** REQ-001 (criteria 1, 3), REQ-015 (criterion 1); TASK-002, TASK-007. **Evidence:** S, W.

**Preconditions/data:** Flask test client for a fresh synthetic app; wireframe W available for visual comparison; viewport 480×360 for layout review.

1. GET the proposed `/sign-in` route with scripts disabled; record status, `Content-Type` and response HTML.
2. Inspect rendered DOM/viewport for white background, narrow horizontally centered dark-bordered panel, bold “Sign in” heading, labeled “Email address” input with `name@example.com` placeholder, and labeled “Password” input.
3. Inspect the blue underlined “Forgot password?” link below password, wide dark “Sign in” button and muted “No account? Contact your administrator.” footer.

**Expected:** (1) Successful HTML page response containing the content without client-side rendering. (2) All listed input/heading/panel features are present and match W's visible arrangement, without claiming pixel-perfect recovery-page layout. (3) Link/button/footer appear as pictured; this test alone does not prove that they work or that contrast meets a standard.

### TC-002 — Live password masking, including empty boundary
**Links:** REQ-001 (criterion 2); TASK-002, TASK-007. **Evidence:** S, W.

**Preconditions/data:** Rendered sign-in page in a browser; synthetic `old-Secret-1`; empty string as initial value.

1. Check the empty password field's DOM type, associated label and initial value.
2. Enter `old-Secret-1` and inspect on-screen characters and DOM input type without submitting.
3. Clear the field and repeat with a single character.

**Expected:** (1) Empty labeled input has `type=password`. (2) Entered characters are concealed on screen and type remains `password`; W's masked sample alone would not establish this live behavior. (3) Both empty and one-character boundaries remain password-type; no test requires accepting a one-character password.

### TC-003 — Recovery link reaches a usable request form
**Links:** REQ-002 (criterion 1), REQ-014 (criterion 1), REQ-015 (criterion 1); TASK-002, TASK-004, TASK-007. **Evidence:** S, W, K.

**Preconditions/data:** Fresh app and Flask/browser client; synthetic account `alex@example.test`; valid form CSRF token for the proposed protected POST.

1. GET `/sign-in`, follow the actual “Forgot password?” `href`, and record both response statuses and the destination route.
2. Without JavaScript, inspect destination HTML for a labeled email identifier input and submit action.
3. POST `alex@example.test` with a valid CSRF token through that form and follow the response's redirect if any.

**Expected:** (1) The link leads to the proposed `/forgot-password` request entry, not a nonexistent route or client-only transition. (2) A server-provided HTML form allows the email identifier assumed from W; no recovery screen was pictured. (3) Valid submission returns a neutral confirmation (HTML response or redirect followed by HTML), not a token/link or sink contents in the visitor response.

### TC-004 — Known, unknown and quota-suppressed responses disclose no identity
**Links:** REQ-003 (criterion 1); TASK-003, TASK-004, TASK-007. **Evidence:** S, K.

**Preconditions/data:** Isolated synthetic apps or resettable fixtures at the same aware-UTC instant; `alex@example.test` known, `missing@example.test` unknown, and a known account with five eligible requests already accepted; equivalent valid email-form submissions and valid CSRF tokens. Compare visitor outputs separately from private sink effects.

1. POST each identifier to the same proposed request route with identical valid form conditions; do not follow redirects yet.
2. Compare status, `Location` (if any), response body, and account-independent headers (`Content-Type` and applicable cache/referrer headers); then follow redirects under equivalent conditions and compare final statuses and bodies.
3. Search each visitor body/header for account existence, quota details, reset token/URL and private sink contents.

**Expected:** (1–2) Known, unknown and over-limit requests have identical visitor-facing status, redirect destination, final rendered body and account-independent header values; per-client CSRF/session cookies, if generated, are not interpreted as identity disclosures or compared as fixed values. (3) None exposes existence, suppression or a link. K mandates known/unknown equivalence; the over-limit neutral HTTP behavior is the approved design proposal. Do not assert equal processing time.

### TC-005 — Five allowed, sixth suppressed inside rolling hour
**Links:** REQ-004 (criterion 1); TASK-003, TASK-007. **Evidence:** S, K.

**Preconditions/data:** Fresh `alex@example.test` account; clock fixed at `2026-01-01T12:00:00Z`; fixture-accessible sink and issuance ledger; valid requests with CSRF tokens.

1. Send five reset requests for Alex at `t`, recording private sink link count and owner after each.
2. Set clock to `t + 59 minutes 59.999 seconds` and send the sixth request.
3. Compare sink link count and issuance-ledger accepted timestamps before and after the sixth; inspect public response only for neutrality.

**Expected:** (1) Five distinct owner-addressed reset-link records and five accepted issuances. (2–3) No sixth link or accepted issuance appears while the oldest is still inside the rolling hour; visitor output does not reveal quota or existence.

### TC-006 — Eligibility resumes at the one-hour boundary
**Links:** REQ-004 (criterion 2); TASK-003, TASK-007. **Evidence:** S, K.

**Preconditions/data:** Five eligible requests for `alex@example.test` accepted at the same aware-UTC `t`; the sixth was suppressed before the boundary; fixture-controlled clock.

1. Set clock to exactly `t + 1 hour` and submit another valid request for Alex.
2. Inspect Alex's sink records and accepted issuance timestamps.

**Expected:** (1–2) One new link is recorded (total six) and its issue time is `t + 1 hour`; old entries at or before the cutoff no longer block an eligible request. This tests the approved rolling-hour interpretation rather than an unspecified calendar-hour bucket.

### TC-007 — Eligible link is private and sink-inspectable
**Links:** REQ-005 (criterion 1), REQ-012 (criterion 1); TASK-001, TASK-003, TASK-007. **Evidence:** S, K.

**Preconditions/data:** Fresh app, known synthetic `alex@example.test`, private injected sink, fixed link origin and time; no external delivery adapter.

1. Submit one eligible request with a valid CSRF token and capture the visitor response.
2. Inspect the injected sink directly in the test fixture for its new record and parse its link.
3. Inspect stored grant state and public response separately.

**Expected:** (1) Neutral visitor response contains no reset URL or token. (2) Exactly one owner-addressed reset-link record contains a URL for the proposed reset route using the configured origin, not a requester-supplied Host; it is available without sending email. (3) Grant state has the token digest/account ID/UTC issue time but no raw token, and no public sink-inspection endpoint is needed. Digest-only state and fixed origin are design proposals.

### TC-008 — Unknown identifier never produces account link
**Links:** REQ-005 (criterion 2); TASK-003, TASK-007. **Evidence:** S, K.

**Preconditions/data:** `missing@example.test` absent from isolated synthetic store; private sink initially empty; valid CSRF token.

1. Submit a reset request for `missing@example.test`.
2. Inspect private sink and grant/issuance state, then compare visitor response using TC-004's known-account baseline.

**Expected:** (1–2) No account-specific link, grant or existing-account issuance is recorded; visitor-facing fields are identical to the equivalent known-account request. No unregistered address is notified.

### TC-009 — Sequential reuse cannot complete another reset
**Links:** REQ-006 (criterion 1); TASK-005, TASK-006, TASK-008. **Evidence:** S, K.

**Preconditions/data:** One unused link for synthetic Alex, unexpired injected time, prior `old-Secret-1`; first replacement `new-Secret-2`, replay candidate `other-Secret-3`; valid form CSRF tokens.

1. GET the link, POST `new-Secret-2` with a valid CSRF token and capture redirect and private notice count.
2. POST `other-Secret-3` with the *same link* and fresh valid CSRF token while time remains before expiry.
3. Check resulting page/redirect, credential verification and notice count.

**Expected:** (1) One guarded completion leads to the proposed token-free completion route; the GET does not consume the grant. (2–3) Reuse yields non-success unusable-link guidance, never a second completion redirect; `new-Secret-2` still verifies, `other-Secret-3` does not, and no second successful-reset notice is recorded. Exact failure status/copy is not prescribed.

### TC-010 — Simultaneous replay has one winner
**Links:** REQ-006 (criterion 1); TASK-005, TASK-008. **Evidence:** S, K.

**Preconditions/data:** One unused link at an unexpired frozen time; two independent CSRF-valid clients/worker invocations synchronized to attempt the same grant with distinct nonempty replacement values; one guarded in-process store.

1. Synchronize two completion POSTs to overlap, then release both.
2. Record each response and inspect grant, password, session generation and sink.

**Expected:** (1–2) At most one request reports success/redirects to token-free completion; exactly one candidate password wins, grant is consumed once and exactly one owner success notice is appended. The losing request neither overwrites the winner nor claims completion. This tests the proposed single-process atomicity, not cross-worker guarantees.

### TC-011 — Valid immediately before expiry; read-only link GET
**Links:** REQ-007 (criterion 1); TASK-005, TASK-006, TASK-008. **Evidence:** S, K.

**Preconditions/data:** Fresh link issued at aware-UTC `t = 2026-01-01T12:00:00Z`; frozen clock at `t + 29 minutes 59.999 seconds`; valid CSRF token and `new-Secret-2`.

1. GET the token URL twice and inspect both HTML responses and grant state after each.
2. Submit the labeled new-password form at the same frozen instant.
3. Inspect completion response, password and notification state.

**Expected:** (1) Successful HTML new-password form on both visits, with unchanged unconsumed grant; visiting does not reset the expiry clock. (2–3) POST strictly before `t + 30 minutes` remains eligible, completes once and changes the password with one owner notice. A GET alone never produces a success notice.

### TC-012 — Exact thirty-minute boundary is expired
**Links:** REQ-007 (criterion 2); TASK-005, TASK-006, TASK-008. **Evidence:** S, K.

**Preconditions/data:** Separate fresh unused link issued at `t`; freeze clock exactly at `t + 30 minutes`; original password and session generation recorded; valid CSRF token.

1. GET the link at the boundary.
2. Attempt POST with `new-Secret-2` at exactly the boundary, including a form obtained just before expiry if necessary to exercise POST revalidation.
3. Inspect response, credentials, session generation, grant and notice sink.

**Expected:** (1) Generic unusable/expired guidance, not an eligible new-password action. (2–3) No success page/redirect and no password, session generation or success-notice mutation; a form opened earlier does not bypass expiration. No exact failure status or message is mandated.

### TC-013 — Later than expiry and invalid token stay unusable
**Links:** REQ-007 (criterion 2); TASK-005, TASK-006, TASK-008. **Evidence:** S, K.

**Preconditions/data:** Unused link at `t`, clock `t + 30 minutes + 1 microsecond`; separately a syntactically plausible unknown token; original password and notice count.

1. GET and POST the expired link, then GET and POST the unknown token using the route's valid-form/CSRF conditions.
2. Inspect each visitor outcome, grant state, password and sink; follow the offered new-request path.

**Expected:** (1) Both links show generic unusable guidance with a functioning new-request route, never completion guidance; POST cannot replace a password. (2) No notice or account-specific token/sink data is exposed and original state remains unchanged. Generic invalid-link presentation is an approved UI/solution proposal, not pictured.

### TC-014 — Replacement password is the only accepted one
**Links:** REQ-008 (criterion 1); TASK-005, TASK-006, TASK-008. **Evidence:** S, K.

**Preconditions/data:** Synthetic Alex with `old-Secret-1`, valid unused link, time strictly before expiry; submitted distinct nonempty `new-Secret-2`.

1. Complete reset with `new-Secret-2` via protected POST.
2. Verify credentials with the synthetic account verifier or proposed minimal sign-in path using each candidate, in isolated subsequent attempts.

**Expected:** (1) Success-gated token-free completion response. (2) Old credential is rejected and new credential accepted; no test asserts an additional password complexity rule or a general authentication product beyond the proposed synthetic demonstration.

### TC-015 — All previously authenticated clients lose access
**Links:** REQ-009 (criterion 1); TASK-002, TASK-005, TASK-008. **Evidence:** S, K.

**Preconditions/data:** Two independent clients A and B signed into one synthetic Alex account with `old-Secret-1`, both able to pass the proposed authenticated demonstration guard; optional client C belongs to separate synthetic Bea; valid reset link and time.

1. Before reset, request guarded resource from A, B and optionally C and record status/redirect/body.
2. Complete Alex's reset from a separate client with `new-Secret-2`.
3. Revisit the guarded resource from A and B without logging in again; compare with C and, separately, a fresh successful Alex login using the new credential.

**Expected:** (1) A and B independently access the guard before reset. (2–3) Neither prior Alex session is authenticated afterward: guard denies access or redirects each to sign-in without protected content. Bea's unrelated session remains valid if used; a new valid Alex login may access the guard. This verifies server-side per-request revocation, not just removal of the resetting client's cookie.

### TC-016 — One owner notification per successful reset only
**Links:** REQ-010 (criterion 1), REQ-012 (criterion 1); TASK-005, TASK-008. **Evidence:** S, K.

**Preconditions/data:** Fresh synthetic Alex, private sink and eligible link; separate later eligible link for a second successful reset, valid CSRF and frozen eligible time for each.

1. Complete the first reset; inspect private sink notice fields and recipient.
2. Attempt used-link and invalid/expired-link completions; inspect notice count.
3. Complete a different eligible link successfully and inspect notice count again.

**Expected:** (1) Exactly one new success notice addressed to Alex's synthetic owner identity, containing no raw reset token or password. (2) Rejected attempts create no success notices. (3) Exactly one additional owner notice exists for the second **successful** reset; records are available in memory with no external delivery. K mandates owner notification; sink is S's prototype mechanism.

### TC-017 — Synthetic fixture and fresh-instance isolation
**Links:** REQ-011 (criterion 1); TASK-001, TASK-003, TASK-007. **Evidence:** S.

**Preconditions/data:** Two independent app factories A and B; configured synthetic `alex@example.test` only in A, `bea@example.test` only in B; separate injected clocks, ledgers and sinks.

1. Enumerate configured test fixture accounts and delivery adapters; ensure no real user credential or production account connector is configured.
2. Request a link for Alex in A and inspect both app states, then construct a fresh third app C and inspect its accounts/ledger/sink.

**Expected:** (1) All recovery participants are explicit synthetic fixtures; no real-account lookup/delivery dependency is called. (2) A's link/account/quota state is absent from B and C unless explicitly supplied as a fixture. This verifies prototype isolation, not production-grade account storage.

### TC-018 — Links and notices are in-memory, not sent
**Links:** REQ-012 (criterion 1); TASK-001, TASK-003, TASK-005, TASK-007, TASK-008. **Evidence:** S.

**Preconditions/data:** Inject a recording in-memory sink and a fail-if-called external delivery stub; synthetic Alex and a valid link.

1. Request an eligible reset, inspect sink directly through the fixture, then complete the reset and inspect it again.
2. Assert delivery-stub call count and attempt to locate any public sink/list endpoint in the Flask routes.

**Expected:** (1) Private memory contains a link record then an owner success-notice record; both are test-inspectable. (2) Zero external deliveries and no public endpoint listing tokens or notifications. In-memory records are not a real inbox.

### TC-019 — Python 3.12/Flask and server-rendered routes
**Links:** REQ-013 (criterion 1), REQ-014 (criterion 1), REQ-015 (criterion 1); TASK-001, TASK-002, TASK-004, TASK-006, TASK-007, TASK-008. **Evidence:** S.

**Preconditions/data:** Python **3.12** environment with declared dependencies installed; app factory and Flask test client; a fixture-generated valid link plus an invalid link; JavaScript disabled.

1. Under `python3.12`, import and instantiate the Flask app and start the project's declared pytest invocation; record interpreter and framework identity.
2. GET proposed sign-in, request, neutral confirmation (via valid POST), new-password, invalid-link and success-gated completion routes through Flask test client; record statuses, redirects and final HTML bodies.
3. Inspect raw responses for each page's headings, labeled form controls or guidance without running scripts; request direct completion without a success state.

**Expected:** (1) App and test runner execute under Python 3.12 with declared Flask/pytest dependencies; do not substitute a different interpreter. (2–3) Flask handles route requests and supplies HTML content on the server (successful page GETs yield HTML; valid request/complete POSTs can redirect as proposed); direct completion does **not** falsely report success. No external SPA renderer is necessary. This is a proposed test, not a report of a run.

### TC-020 — Automated pytest suite covers required scenarios
**Links:** REQ-016 (criterion 1); TASK-007, TASK-008. **Evidence:** S, W, K.

**Preconditions/data:** Complete proposed implementation and declared Python 3.12/pytest test dependencies; isolated synthetic fixtures.

1. Inspect test names/assertions and map them to the pictured entry, neutral known/unknown response, five/six/hour limit, private link issuance, single use, thirty-minute boundary, password replacement, all-session sign-out and owner notice.
2. Run `python3.12 -m pytest -q` in the future implementation; record collection, exit status and any failures.

**Expected:** (1) Each named behavior has an executable pytest assertion (not just a comment or placeholder), including private sink and multi-client/clock boundaries. (2) Tests are collected and pass with zero exit status when the implementation satisfies the cases; failures are reported, not silently excluded. **No test run is claimed here.**

### TC-021 — Production limitations are explicit
**Links:** REQ-017 (criterion 1); TASK-009. **Evidence:** S.

**Preconditions/data:** Delivered project's documentation.

1. Read its production-limitations section and locate statements about account identity, delivery, storage and operating model.
2. Check the text for any conflicting claim that the prototype provides real user accounts, durable/shared state, real email delivery or production readiness.

**Expected:** (1) Documentation identifies synthetic accounts and private in-memory notifications as unsuitable substitutes for production account and delivery systems; explains restart loss, process-local/multi-worker limitations and absence of external delivery. (2) No conflicting production-readiness assertion. This is a documentation inspection, not an implementation or deployed-service test.

### TC-022 — Read-only non-deployment scope inspection
**Links:** REQ-018 (criterion 1); TASK-009. **Evidence:** S.

**Preconditions/data:** Delivered project diff/file tree, workflow files and available task-action record; inspection permissions only, no deployment credentials/actions.

1. Inspect delivered changes and `.github/workflows/` (if present) for added deployment workflows, jobs, scripts or deployment configurations; inspect documented commands for executed deployment actions.
2. Review available action record for any performed deployment; report separately if evidence of execution is unavailable.

**Expected:** (1) No deployment workflow was added and no deployment action appears in the available delivery record; do not execute one as a test. (2) If the action record is unavailable, explicitly report the unverified *execution-history* aspect rather than infer it from repository files. No current deployment absence is claimed by this draft.

### TC-023 — CSRF and invalid sign-in do not bypass proposed guards
**Links:** REQ-002 (criterion 1), REQ-009 (criterion 1); TASK-002, TASK-004, TASK-005, TASK-006. **Evidence:** S, K.

**Preconditions/data:** Synthetic Alex, independent unauthenticated client, proposed CSRF-protected mutation routes and guarded resource; test only approved minimal sign-in, not an unspecified production auth flow.

1. Attempt synthetic sign-in with wrong password and request the guarded resource.
2. POST valid-form reset request without CSRF token and with an invalid token; compare sink/ledger state before and after.
3. Obtain a valid reset link via fixture; POST completion without/with invalid CSRF and compare grant, password, session generation and notices before and after.

**Expected:** (1) Wrong credentials never create a valid guarded session. (2) Missing/invalid CSRF is rejected (non-success HTTP outcome, not a neutral successful POST) without issuance. (3) Missing/invalid CSRF cannot consume grant, change credentials, revoke sessions or create notice. CSRF and guarded sign-in are approved design **security proposals**, not requirements stated by K or observed in W; exact rejection status is not prescribed.

### TC-024 — Empty/malformed identifier and empty replacement
**Links:** REQ-002 (criterion 1), REQ-008 (criterion 1); TASK-004, TASK-005, TASK-006. **Evidence:** S, W, K.

**Preconditions/data:** Server-rendered request/reset forms; valid CSRF; request inputs `""`, `"not-an-email"` and valid `alex@example.test`; reset input `""` for an unused valid link. Email is the pictured/approved **assumed** identifier, not a standard-prescribed validator; no extra password rules are inferred.

1. Submit empty and malformed request identifiers, then the valid identifier as control.
2. With an eligible link, submit empty new-password input; inspect returned HTML, grant, credential and notice state, then submit a nonempty value as control.

**Expected:** (1) Invalid form inputs receive accessible field-associated errors or are blocked by native form validation without recording links; the valid control can proceed neutrally. (2) Empty replacement never completes, changes password/session generation or records success notice; the still-valid link remains usable for a nonempty replacement strictly before expiry. No mandated email-error wording or password-strength threshold is invented.

### TC-025 — Sink/update failure must not be reported as success
**Links:** REQ-006 (criterion 1), REQ-008 (criterion 1), REQ-009 (criterion 1), REQ-010 (criterion 1); TASK-001, TASK-005, TASK-006, TASK-008. **Evidence:** S, K.

**Preconditions/data:** Synthetic Alex, active client sessions, unexpired unused link, recorded old password, grant and session generation; injected sink whose success-notice append raises an error, plus separately injected controlled account-update failure; no real delivery.

1. Attempt reset with `new-Secret-2` while success-notice append fails; capture HTTP response and state.
2. Restore sink; separately attempt with `other-Secret-3` while the controlled account update fails; inspect state.
3. Remove fault, retry on a still-eligible grant if applicable and verify eventual outcome.

**Expected:** (1–2) Neither failure returns a success-gated `/reset/complete` redirect/body; no false owner notice exists. Guarded transaction has no partially reported success: old credential, session validity/generation and grant/notice consistency are retained or explicitly rolled back for safe retry. (3) A successful retry, if before expiry, commits exactly one password change, revocation and owner notice. This is the approved design's failure-atomicity proposal; a sink failure cannot masquerade as delivery success.

### TC-026 — Token privacy, completion gating and no public sink
**Links:** REQ-005 (criterion 1), REQ-006 (criterion 1), REQ-015 (criterion 1); TASK-003, TASK-005, TASK-006. **Evidence:** S, K.

**Preconditions/data:** Synthetic fixture-issued valid token, configured fixed origin, optional requester-controlled Host header, private sink fixture and an unauthenticated visitor.

1. Request a link with an attacker-controlled Host, inspect sink URL and response; GET token-bearing page and inspect headers and HTML resources.
2. Complete reset, inspect redirect target, response body and completion page; request completion URL directly in a fresh client without success state.
3. Enumerate Flask routes exposed to visitor and request any suspected public sink/token listing.

**Expected:** (1) Generated link uses configured origin, not Host; token-bearing response has no-store/no-referrer protections and no third-party assets capable of receiving URL referrers; token is not reflected in unrelated public output. (2) Successful redirect URL contains no token; direct completion does not assert success. (3) No public sink/token listing; private link is inspectable only via fixture. These are design privacy protections, **not** independent provisions of K.

### TC-027 — Keyboard, labels, focus, errors and contrast (proposed accessibility)
**Links:** REQ-001 (criteria 1, 3), REQ-002 (criterion 1), REQ-015 (criterion 1); TASK-002, TASK-004, TASK-006, TASK-007, TASK-008. **Evidence:** S, W, K.

**Preconditions/data:** Browser/assistive-technology inspection of pictured sign-in and proposed recovery/request/new-password/invalid/confirmation pages; keyboard only; empty and malformed identifier test values; contrast measurement tool. Accessibility behavior is proposed in approved UI analysis, not established by W.

1. Tab/Shift+Tab and activate link, inputs and buttons using keyboard; inspect logical order, visible focus, control accessible names and programmatic label associations.
2. Trigger empty/malformed form errors and a neutral confirmation/unusable-link transition; inspect field-error association, status announcement and focus placement on new state/heading.
3. Measure contrast for normal/large text, form controls and focus indicators, especially W's blue link and muted footer; record measured ratios against the project's adopted accessibility target (proposed WCAG AA target if approved).

**Expected:** (1) All controls are keyboard-operable, have programmatic names/labels and visible logical focus. (2) Errors are associated/announced and changed confirmation/error state can be found by focus or announcement without exposing account identity. (3) Measured ratios meet the adopted target, or the failing element and ratio are explicitly reported; colors in W alone are **not** evidence of compliance. Do not claim a target was mandated by K.

### TC-028 — Independent per-account quota
**Links:** REQ-004 (criterion 1), REQ-005 (criterion 1); TASK-003, TASK-007. **Evidence:** S, K.

**Preconditions/data:** Two synthetic accounts Alex and Bea, independent ledgers in one app; Alex has five accepted links at `t`, Bea has none; current time inside Alex's first rolling hour.

1. Send Alex's sixth valid request, then Bea's first valid request.
2. Inspect owner-addressed sink counts and separate ledgers.

**Expected:** (1–2) Alex gains no sixth link but Bea gains exactly one; throttling is **per account**, not global, and no public response reveals either account's state.

## Acceptance-criterion coverage index

| Approved criterion | Proposed verifying TC IDs |
| --- | --- |
| REQ-001 · 1 | TC-001, TC-027 |
| REQ-001 · 2 | TC-002 |
| REQ-001 · 3 | TC-001, TC-027 |
| REQ-002 · 1 | TC-003, TC-023, TC-024, TC-027 |
| REQ-003 · 1 | TC-004, TC-008 |
| REQ-004 · 1 | TC-005, TC-028 |
| REQ-004 · 2 | TC-006 |
| REQ-005 · 1 | TC-007, TC-026, TC-028 |
| REQ-005 · 2 | TC-008 |
| REQ-006 · 1 | TC-009, TC-010, TC-025, TC-026 |
| REQ-007 · 1 | TC-011 |
| REQ-007 · 2 | TC-012, TC-013 |
| REQ-008 · 1 | TC-014, TC-024, TC-025 |
| REQ-009 · 1 | TC-015, TC-023, TC-025 |
| REQ-010 · 1 | TC-016, TC-025 |
| REQ-011 · 1 | TC-017 |
| REQ-012 · 1 | TC-007, TC-016, TC-018 |
| REQ-013 · 1 | TC-019 |
| REQ-014 · 1 | TC-003, TC-019 |
| REQ-015 · 1 | TC-001, TC-003, TC-019, TC-026, TC-027 |
| REQ-016 · 1 | TC-020; the executable behavior assertions proposed in TC-001–TC-019 and TC-023–TC-028 |
| REQ-017 · 1 | TC-021 |
| REQ-018 · 1 | TC-022 |

**Coverage summary:** 23/23 approved acceptance criteria, 18/18 approved requirements, and 9/9 approved TASK IDs have at least one proposed case. **Uncovered criteria: none.** Coverage is a design trace, not evidence of execution or conformance. TC-022's execution-history assertion remains conditional on an available action record; absence of that record is a validation limitation to report, not proof of non-deployment. No unpictured recovery layout, password-strength policy, live authentication behavior or real notification delivery is asserted as observed.
