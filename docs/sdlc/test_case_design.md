# Proposed password-reset test cases

**Status:** Design only. None of these cases has been executed or passed; no implementation or deployment is asserted. Criterion ordinals below are 1-based within each approved requirement, not new identifiers. The proposed routes and technical controls come from the approved solution design, not from an observed implementation.

## Evidence and interpretation

| Key | Exact source reference | Boundary |
| --- | --- | --- |
| S | `{"source_type":"user_story","source_id":"user_story","locator":"characters 0-16000"}` | Python 3.12, Flask, server-rendered pages, pytest, synthetic accounts, in-memory notification sink, documented limitations and non-deployment. |
| W | `{"source_type":"attachment","source_id":"23b23ac1-a532-42fc-9973-51a1d4954aa6","locator":"sign-in-wireframe.png"}` | A white 480×360 image shows a narrow horizontally centered dark-bordered panel, bold “Sign in,” labeled “Email address” with `name@example.com`, labeled masked-sample Password field, blue underlined “Forgot password?” below it, wide dark button and muted “No account? Contact your administrator.” footer. Neither working sign-in nor recovery screens are pictured. |
| K | `{"source_type":"knowledge","source_id":"bcf198d8-a628-401c-8641-b726fcbd00b5","locator":"v1, chunk 0"}` | Curated account-recovery standard: entry link, identical existing/unknown responses, five requests per account per hour, single-use links expiring at 30 minutes, all-session sign-out and post-success owner notification. |

Approved `requirements_analysis`, `ui_analysis` and `solution_design` establish the **REQ-001–REQ-018** and **TASK-001–TASK-009** links, but returned no standalone `stage_output` source-reference locator; none is invented. Test-only synthetic identities `alex@example.test`, `bea@example.test`, `missing@example.test` and passwords `old-Secret-1`, `new-Secret-2`, `other-Secret-3` are fixtures, not production identities or password policy. Proposed design uses isolated app factories, injected aware-UTC time, private fixture-only sink and independent clients. Proposed CSRF, fixed link origin, atomic rollback, token-page protections and synthetic guarded sign-in are not separate mandates of K. The wireframe does not establish live masking or accessibility conformance.

## Cases

### TC-001 — Pictured sign-in content and layout
**Trace:** REQ-001 criteria 1, 3; REQ-015 criterion 1 · TASK-002, TASK-007 · S, W.  
**Preconditions/data:** Fresh synthetic Flask app/client; W for visual comparison; 480×360 viewport.
1. GET proposed `/sign-in` with scripts disabled; record status, Content-Type and raw HTML.
2. Inspect viewport and DOM for white background, narrow centered dark-bordered panel, bold “Sign in,” labeled “Email address” with `name@example.com` placeholder and labeled “Password.”
3. Inspect blue underlined “Forgot password?” below password, wide dark “Sign in” button and muted administrator-contact footer.

**Expected:** (1) Successful server HTML response without client rendering. (2) Listed panel/heading/field features match W's visible arrangement; no unpictured recovery layout is implied. (3) Pictured link, button and footer appear; functionality and accessibility need separate tests.

### TC-002 — Live password masking and empty boundary
**Trace:** REQ-001 criterion 2 · TASK-002, TASK-007 · S, W.  
**Preconditions/data:** Sign-in page in browser; initially empty password, `old-Secret-1`.
1. Inspect empty Password field DOM type, value and label.
2. Enter `old-Secret-1` without submission; inspect on-screen characters and input type.
3. Clear and enter one character; inspect again.

**Expected:** (1) Empty labeled input has `type=password`. (2) Characters are concealed while type stays password; W's masked sample alone cannot prove this. (3) Empty and one-character boundaries remain password-type; acceptance of one-character passwords is not prescribed.

### TC-003 — Recovery link reaches a usable request form
**Trace:** REQ-002, REQ-014, REQ-015 criterion 1 each · TASK-002, TASK-004, TASK-007 · S, W, K.  
**Preconditions/data:** Fresh app/client, `alex@example.test`, valid proposed CSRF token.
1. GET `/sign-in`, follow actual “Forgot password?” href and record destination/status.
2. With scripts disabled inspect destination HTML for labeled email identifier and submit control.
3. POST Alex's identifier with valid form protection and follow any redirect.

**Expected:** (1) Link reaches proposed `/forgot-password`, not a missing route or client-only transition. (2) Server-provided form accepts assumed email identifier; no recovery mockup exists. (3) Visitor sees neutral confirmation, not token, URL or sink content.

### TC-004 — Known, unknown and quota-suppressed responses remain neutral
**Trace:** REQ-003 criterion 1 · TASK-003, TASK-004, TASK-007 · S, K.  
**Preconditions/data:** Isolated fixtures at same aware-UTC instant: known `alex@example.test`, unknown `missing@example.test`, known account already at five accepted requests; equivalent valid forms and CSRF.
1. POST each identifier under equivalent conditions without following redirects.
2. Compare status, Location, body and account-independent Content-Type/cache/referrer headers; follow redirects and compare final statuses/bodies.
3. Inspect visitor headers/bodies for account existence, quota, token, URL or sink contents.

**Expected:** (1–2) Identical visitor-facing status, destination, final body and account-independent header values. Independently generated cookies/CSRF values need not be byte-identical. (3) No disclosure. Known/unknown equivalence is required by K; over-limit neutrality is proposed design. Equal processing time is not asserted.

### TC-005 — Five accepted, sixth suppressed inside rolling hour
**Trace:** REQ-004 criterion 1 · TASK-003, TASK-007 · S, K.  
**Preconditions/data:** Fresh synthetic Alex, fixed `t=2026-01-01T12:00:00Z`, private sink/ledger, CSRF-valid forms.
1. Submit five requests at `t`, counting owner-addressed links and accepted timestamps.
2. Advance to `t+59m59.999s`; submit sixth.
3. Compare private counts/ledger before and after; check public neutrality.

**Expected:** (1) Five distinct owner links and five issuances. (2–3) No sixth link or accepted timestamp within rolling hour; visitor output reveals no quota/identity.

### TC-006 — Eligibility resumes at one-hour boundary
**Trace:** REQ-004 criterion 2 · TASK-003, TASK-007 · S, K.  
**Preconditions/data:** Five Alex requests accepted at aware-UTC `t`, sixth suppressed before cutoff; injected clock.
1. At exactly `t+1h`, submit a valid request.
2. Inspect Alex's sink links and accepted issue times.

**Expected:** (1–2) One fresh link (six total) issued at `t+1h`; old entries at/before cutoff do not block it. Rolling-hour, not calendar-hour, interpretation comes from approved design.

### TC-007 — Eligible link is private and sink-inspectable
**Trace:** REQ-005, REQ-012 criterion 1 each · TASK-001, TASK-003, TASK-007 · S, K.  
**Preconditions/data:** Fresh app, synthetic Alex, injected private memory sink, configured origin/time, no external delivery.
1. Submit eligible request with valid CSRF; capture visitor output.
2. Inspect fixture sink for owner-addressed link and parse its URL.
3. Compare private grant record with public response.

**Expected:** (1) Neutral visitor response lacks link/token. (2) Exactly one owner link uses configured origin and is available without external mail. (3) Proposed grant has digest/account ID/UTC issue time, not raw token; no public sink route. Digest-only storage/fixed origin are design choices.

### TC-008 — Unknown identifier produces no account link
**Trace:** REQ-003 criterion 1; REQ-005 criterion 2 · TASK-003, TASK-007 · S, K.  
**Preconditions/data:** `missing@example.test` absent from isolated store, empty sink, valid CSRF.
1. Request reset for missing identifier.
2. Inspect sink/grants/accepted issuances; compare visitor response with TC-004 known baseline.

**Expected:** (1–2) No account link, grant, accepted issuance or notification to unknown address; visitor output matches equivalent known request.

### TC-009 — Sequential reuse cannot reset twice
**Trace:** REQ-006 criterion 1 · TASK-005, TASK-006, TASK-008 · S, K.  
**Preconditions/data:** Unexpired unused Alex link, `old-Secret-1`, distinct `new-Secret-2`/`other-Secret-3`, valid CSRF.
1. GET link, POST `new-Secret-2`; record redirect and notice count.
2. Before expiry POST `other-Secret-3` with same link and fresh valid CSRF.
3. Check response, both credential candidates and notice count.

**Expected:** (1) First reset reaches token-free completion; GET did not consume link. (2–3) Reuse yields non-success unusable guidance, not a second completion; first new password still verifies, replay candidate fails and no second notice exists.

### TC-010 — Simultaneous replay has one winner
**Trace:** REQ-006 criterion 1 · TASK-005, TASK-008 · S, K.  
**Preconditions/data:** One eligible link, two independent CSRF-valid clients/invocations with distinct nonempty replacements, guarded in-process store.
1. Synchronize two overlapping completion POSTs and release both.
2. Inspect both responses, grant, password, session generation and sink.

**Expected:** (1–2) At most one success and exactly one winning password, one consumed grant and one owner notice. Losing request neither overwrites nor claims completion. This tests single-process atomicity, not multi-worker durability.

### TC-011 — Valid just before expiry; read-only GET
**Trace:** REQ-007 criterion 1 · TASK-005, TASK-006, TASK-008 · S, K.  
**Preconditions/data:** Fresh link at `t=2026-01-01T12:00:00Z`; frozen at `t+29m59.999s`; valid CSRF and `new-Secret-2`.
1. GET link twice; inspect HTML, grant and issue time.
2. POST new-password form at same instant.
3. Inspect completion, credential and notices.

**Expected:** (1) Both GETs return usable HTML form without consuming grant or extending expiry. (2–3) POST strictly before `t+30m` completes once, changes credential and creates one owner notice; GET alone creates none.

### TC-012 — Exact thirty-minute boundary is expired
**Trace:** REQ-007 criterion 2 · TASK-005, TASK-006, TASK-008 · S, K.  
**Preconditions/data:** Fresh unused link at `t`, clock exactly `t+30m`, recorded old password/session generation, valid CSRF.
1. GET link at cutoff.
2. POST `new-Secret-2` at cutoff, including using pre-opened form if needed.
3. Inspect visitor response, password, session generation, grant and sink.

**Expected:** (1) Generic unusable guidance, not eligible form. (2–3) No success state, credential/session change or owner success notice; a form opened before cutoff cannot bypass POST recheck. No exact failure status/copy specified.

### TC-013 — Later expiry and unknown token stay unusable
**Trace:** REQ-007 criterion 2 · TASK-005, TASK-006, TASK-008 · S, K.  
**Preconditions/data:** Unused link at `t`, clock `t+30m+1µs`, plausible unknown token, old password and notice count.
1. GET and CSRF-valid POST expired link, then unknown token.
2. Inspect outcomes and state; follow new-request path.

**Expected:** (1) Both yield generic unusable guidance and working new-request route, never completion or password replacement. (2) No state mutation or account-specific leak. Generic invalid-link screen is a proposal, not pictured.

### TC-014 — Replacement password is the only accepted credential
**Trace:** REQ-008 criterion 1 · TASK-005, TASK-006, TASK-008 · S, K.  
**Preconditions/data:** Synthetic Alex with `old-Secret-1`, eligible unused link, distinct nonempty `new-Secret-2`.
1. Complete protected reset using new password.
2. Verify old and new credentials in separate synthetic-verifier or minimal sign-in attempts.

**Expected:** (1) Token-free gated completion. (2) Old password rejected and new accepted; no added strength rule or full authentication product inferred.

### TC-015 — All previously authenticated clients lose access
**Trace:** REQ-009 criterion 1 · TASK-002, TASK-005, TASK-008 · S, K.  
**Preconditions/data:** Independent clients A/B signed into synthetic Alex, able to access guarded resource; optional Bea client C; valid link.
1. Access guard from A/B and optional C before reset.
2. Reset Alex via separate client with `new-Secret-2`.
3. Revisit guard from A/B without logging in; optionally check C and fresh Alex login.

**Expected:** (1) A/B each access guard initially. (2–3) Both old Alex sessions denied/redirected without protected content; Bea remains valid if tested, fresh Alex login using new password may access. Clearing only resetting client's cookie is insufficient.

### TC-016 — One owner notice per successful reset only
**Trace:** REQ-010, REQ-012 criterion 1 each · TASK-005, TASK-008 · S, K.  
**Preconditions/data:** Synthetic Alex, private sink, two separate eligible links at valid times, CSRF-valid completion.
1. Complete first reset; inspect notice recipient and contents.
2. Attempt used, invalid and expired links; recount.
3. Complete second eligible link; recount.

**Expected:** (1) One owner-addressed success record, no raw token/password. (2) No new success notice for rejected attempts. (3) One further owner record for second success, inspectable only in memory; external delivery not assumed.

### TC-017 — Synthetic fixture and fresh-instance isolation
**Trace:** REQ-011 criterion 1 · TASK-001, TASK-003, TASK-007 · S.  
**Preconditions/data:** App A has only synthetic Alex, B only synthetic Bea; separate clocks/ledgers/sinks.
1. Inspect fixture accounts/adapters for real credentials or production connector.
2. Issue Alex link in A, inspect A/B; construct fresh C and inspect state.

**Expected:** (1) Only synthetic participants, no real lookup/delivery call. (2) A's account/link/quota absent in B/C unless explicitly provided; not a production storage claim.

### TC-018 — Links and notices are in-memory, not sent
**Trace:** REQ-012 criterion 1 · TASK-001, TASK-003, TASK-005, TASK-007, TASK-008 · S.  
**Preconditions/data:** Recording memory sink and fail-if-called external-delivery stub; synthetic Alex and valid link.
1. Issue request, inspect fixture sink; complete reset and inspect again.
2. Assert external stub calls and inspect public Flask routes for sink/token-list endpoint.

**Expected:** (1) Private memory records link and owner notice. (2) Zero external calls and no public listing; memory sink is not a real inbox.

### TC-019 — Python 3.12/Flask and server-rendered routes
**Trace:** REQ-013, REQ-014, REQ-015 criterion 1 each · TASK-001, TASK-002, TASK-004, TASK-006, TASK-007, TASK-008 · S.  
**Preconditions/data:** Python 3.12 with declared dependencies, Flask app/client, valid/invalid fixture links; scripts disabled.
1. Under `python3.12` instantiate app, start declared pytest invocation; record interpreter/framework.
2. Request proposed sign-in, request, neutral confirmation, new-password, invalid-link and gated completion routes; record statuses, redirects and HTML.
3. Inspect raw headings/form labels/guidance without scripts; request completion directly without success state.

**Expected:** (1) App and test runner execute under Python 3.12 with declared Flask/pytest dependencies, not a substitute interpreter. (2–3) Flask handles routes and supplies server HTML; proposed POST redirects may occur; direct completion cannot falsely claim success.

### TC-020 — Automated pytest suite covers required scenarios
**Trace:** REQ-016 criterion 1 · TASK-007, TASK-008 · S, W, K.  
**Preconditions/data:** Future complete implementation, Python 3.12/pytest dependencies and synthetic fixtures; not run now.
1. Inspect executable assertions for pictured entry, neutral known/unknown response, five/six/hour limit, issuance, single use, expiry boundary, replacement, multi-client sign-out and notice.
2. In future implementation run `python3.12 -m pytest -q`; record collection, exit and failures.

**Expected:** (1) Every named behavior has an executable assertion rather than placeholder, including sink/clock/multi-client boundaries. (2) When implementation satisfies cases, tests collect and exit zero; failures are reported, not excluded. No current passing result is claimed.

### TC-021 — Production limitations are explicit
**Trace:** REQ-017 criterion 1 · TASK-009 · S.  
**Preconditions/data:** Future delivered project documentation.
1. Read production-limitations section for accounts, delivery, storage and operating model.
2. Check for conflicting real-account, durable/shared state, real-email or production-readiness claims.

**Expected:** (1) Synthetic accounts/private in-memory notifications identified as unsuitable production substitutes, with restart loss, multi-worker/process-local limits and absent external delivery. (2) No contradictory production claim. Documentation inspection only.

### TC-022 — Read-only non-deployment scope inspection
**Trace:** REQ-018 criterion 1 · TASK-009 · S.  
**Preconditions/data:** Future delivered diff/tree, `.github/workflows/` if present, available action record; read-only inspection, no deployment credentials/actions.
1. Inspect changes, workflows and documented commands for added deployment workflow/job/script/configuration.
2. Review available delivery action history; explicitly record if execution evidence is unavailable.

**Expected:** (1) No deployment workflow added and no performed deployment appears in available record; never deploy as a test. (2) If action history is absent, historical non-deployment is **unverified**, not proven by repo-file absence. No present deployment claim is made.

### TC-023 — CSRF and invalid sign-in do not bypass proposed guards
**Trace:** REQ-002, REQ-009 criterion 1 each · TASK-002, TASK-004, TASK-005, TASK-006 · S, K.  
**Preconditions/data:** Synthetic Alex, unauthenticated client, proposed CSRF-protected routes and guarded resource.
1. Sign in with wrong password and request guard.
2. POST valid-form request with missing/invalid CSRF; compare ledger/sink.
3. Obtain fixture link, POST completion with missing/invalid CSRF; compare grant, password, session and notices.

**Expected:** (1) No guarded session for wrong credentials. (2) Non-success CSRF rejection without link. (3) No grant consumption, credential/session change or notice. CSRF and synthetic sign-in are approved security proposals, not K or W mandates; no precise rejection code prescribed.

### TC-024 — Empty/malformed identifier and empty replacement
**Trace:** REQ-002, REQ-008 criterion 1 each · TASK-004, TASK-005, TASK-006 · S, W, K.  
**Preconditions/data:** Server-rendered forms, valid CSRF; identifiers `""`, `not-an-email`, `alex@example.test`; unused eligible link and empty replacement.
1. Submit empty/malformed request identifiers, then valid control.
2. Submit empty new-password; inspect form, grant, credential, session and notices; retry nonempty value before expiry.

**Expected:** (1) Invalid values get accessible field errors or native validation block with no link; valid control proceeds neutrally. (2) Empty replacement cannot complete/change state; link remains eligible for nonempty retry before expiry. Email validation and nonempty-only password behavior are proposals, not extra standard policy.

### TC-025 — Sink/update failure cannot report success
**Trace:** REQ-006, REQ-008, REQ-009, REQ-010 criterion 1 each · TASK-001, TASK-005, TASK-006, TASK-008 · S, K.  
**Preconditions/data:** Synthetic Alex with active sessions, unexpired link, recorded original credential/grant/generation; injected notice-append and separate account-update faults.
1. Reset to `new-Secret-2` while notice append raises; capture response/state.
2. Restore sink; separately fail account update with `other-Secret-3`; inspect state.
3. Remove fault; if grant remains eligible retry and inspect outcome.

**Expected:** (1–2) No falsely successful completion or notice; guarded transaction retains or safely rolls back credential, session and grant/notice consistency for retry. (3) Any successful retry before expiry commits one replacement, revocation and notice. Fault atomicity/rollback is approved design proposal.

### TC-026 — Token privacy, completion gating and no public sink
**Trace:** REQ-005, REQ-006, REQ-015 criterion 1 each · TASK-003, TASK-005, TASK-006 · S, K.  
**Preconditions/data:** Fixture-issued token, configured origin, optional attacker-controlled Host, private sink, unauthenticated visitor.
1. Request link with hostile Host; inspect fixture URL/public response, token-page headers and HTML resources.
2. Complete reset, inspect redirect/page; directly request completion from fresh client without success state.
3. Inspect public routes and probe any suspected sink/token-list endpoint.

**Expected:** (1) Link uses configured origin; token page proposes no-store/no-referrer and no third-party referrer leaks, no unrelated public token disclosure. (2) Redirect lacks token and direct completion does not claim success. (3) No public sink/listing; fixture-only inspection. These privacy controls are design proposals, not K mandates.

### TC-027 — Keyboard, labels, focus, errors and proposed contrast
**Trace:** REQ-001 criteria 1, 3; REQ-002, REQ-015 criterion 1 each · TASK-002, TASK-004, TASK-006, TASK-007, TASK-008 · S, W, K.  
**Preconditions/data:** Browser/assistive-technology review of pictured sign-in and proposed recovery pages; keyboard, invalid/empty values, contrast meter.
1. Tab/Shift+Tab and activate controls; inspect accessible names, label associations and visible logical focus.
2. Trigger form errors, confirmation and unusable-link transitions; inspect error association, announcement and focus.
3. Measure text/control/focus contrast including blue link/muted footer; record ratios against documented adopted target if any.

**Expected:** (1) Keyboard-operable named/labelled controls and visible logical focus. (2) Discoverable announced/associated errors and transitions without account disclosure. (3) Meet adopted target or report measured failure; if target unresolved, report that rather than assert numeric compliance. WCAG AA is a possible target, not mandated by W or K.

### TC-028 — Independent per-account quota
**Trace:** REQ-004, REQ-005 criterion 1 each · TASK-003, TASK-007 · S, K.  
**Preconditions/data:** Synthetic Alex and Bea in one app with separate ledgers; five accepted Alex links at `t`, Bea zero, clock within Alex's rolling hour.
1. Send Alex's sixth and Bea's first valid request.
2. Inspect owner-addressed sink counts, ledgers and public responses.

**Expected:** (1–2) Alex receives no sixth link; Bea receives exactly one. Limit is per account, not global, and public output does not expose either account's quota or existence.

## Acceptance-criterion coverage

| Approved criterion | Proposed cases |
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
| REQ-016 · 1 | TC-020 |
| REQ-017 · 1 | TC-021 |
| REQ-018 · 1 | TC-022 |

**Coverage:** 23/23 criteria, 18/18 requirements and 9/9 implementation tasks traced to 28 proposed cases. TC-020 proposes inspecting executable assertions for the behavior designed throughout TC-001–TC-019 and TC-023–TC-028. **Uncovered criteria: none.** Coverage is not evidence of execution or compliance. TC-022 cannot establish historical non-deployment without an action record; report unavailable history as a limitation, not as a passed assertion.

## Assumptions and nonblocking questions

- Email is the assumed recovery identifier; recovery layouts/copy and additional password rules are not specified. Synthetic guarded sign-in is a test mechanism proposed to verify session invalidation.
- Process-local state, private sink, clock injection, CSRF, fixed URL origin, guarded rollback, token privacy and accessible state handling are approved design proposals. No real delivery, public token inspection or persistent/multi-worker guarantee is assumed.
- **Questions (nonblocking):** What should the Sign in button do beyond the minimal demonstration? Is particular recovery-page copy/layout or password policy desired? Which accessibility contrast target will be adopted? Will a delivery action record be available to assess non-deployment history?
