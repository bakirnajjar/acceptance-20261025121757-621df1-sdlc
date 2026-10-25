# Acceptance-criterion coverage map

**Scope:** approved REQ-001–REQ-018 acceptance criteria mapped to existing solution-design TASK-001–TASK-009. This is a proposed implementation/test trace, not a report of executed tests or implemented behavior. The criterion number is its **1-based ordinal within the named requirement**, not a newly approved identifier. All 23 criteria are listed; `coverage_dispositions` in the approved solution design is empty, so none is inferred.

## Evidence-reference key

| Key | Exact source reference | What it establishes |
| --- | --- | --- |
| S | `{source_type: user_story, source_id: user_story, locator: "characters 0-16000"}` | Requested prototype, stack, synthetic accounts, in-memory sink, tests, limitations, and non-deployment. |
| W | `{source_type: attachment, source_id: "23b23ac1-a532-42fc-9973-51a1d4954aa6", locator: "sign-in-wireframe.png"}` | **Observed appearance only:** 480×360 white canvas, narrow horizontally centered dark-bordered panel, bold “Sign in,” labeled email with `name@example.com`, labeled password with a masked sample, blue underlined “Forgot password?” below it, wide dark button, and muted administrator-contact footer. It does not establish functional navigation, password masking during entry, authentication, recovery screens, or accessibility behavior. |
| K | `{source_type: knowledge, source_id: "bcf198d8-a628-401c-8641-b726fcbd00b5", locator: "v1, chunk 0"}` | Curated account-recovery standard: recovery link from sign-in, identical existing/unknown responses, five requests per account per hour, single-use links expiring after 30 minutes, all-session sign-out, and owner notification after success. |

The approved requirements, UI analysis, and solution design supply the REQ/TASK identifiers and criterion wording, **but their stage-output responses supplied no independent `stage_output` source-reference locator**. They are linked as predecessors, not cited with an invented locator. Evidence keys below denote exactly the references above, not proof that the proposed tasks have been performed.

## Criterion-to-task trace

| Requirement · criterion ordinal | Approved criterion gist | Applicable approved tasks | Evidence | Observed / specified versus proposed |
| --- | --- | --- | --- | --- |
| REQ-001 · 1 | Render a narrow bordered sign-in panel on white with “Sign in,” labeled “Email address” with `name@example.com` placeholder, and labeled “Password” input. | TASK-002, TASK-007 | S, W | **Observed** pictured content; server-rendered implementation and pytest verification **proposed**. |
| REQ-001 · 2 | Entered password characters are masked. | TASK-002, TASK-007 | S, W | **Observed** masked sample only; live masking and its test **proposed**, not demonstrated by image. |
| REQ-001 · 3 | Show blue underlined “Forgot password?” link, wide dark “Sign in” button, and gray “No account? Contact your administrator.” footer as pictured. | TASK-002, TASK-007 | S, W | **Observed** pictured elements; rendered equivalence and verification **proposed**. |
| REQ-002 · 1 | Following “Forgot password?” lets a visitor submit a reset request for an account identifier. | TASK-002, TASK-004, TASK-007 | S, W, K | **Observed** link affordance; actual navigation, request form, submission and tests **proposed**. |
| REQ-003 · 1 | Equivalent existing-account and unknown-identifier reset requests produce identical visitor-facing responses without revealing existence. | TASK-003, TASK-004, TASK-007 | S, K | **Specified** by standard; neutral service/HTTP handling and comparative tests **proposed**, not pictured. |
| REQ-004 · 1 | After five requests for one synthetic account within an hour, a sixth within that hour issues no sixth link. | TASK-003, TASK-007 | S, K | **Specified** five-per-account-per-hour limit; rolling issuance ledger and boundary test **proposed**. |
| REQ-004 · 2 | After the applicable one-hour period elapses, its earlier five requests do not prevent another eligible request. | TASK-003, TASK-007 | S, K | **Specified** hourly limit; renewed eligibility at the rolling boundary and clock-injected test **proposed**. |
| REQ-005 · 1 | Eligible request for a synthetic account records its reset link in the in-memory sink. | TASK-003, TASK-007 | S | **Story-specified** prototype sink; private link issuance and fixture-only sink inspection **proposed**. |
| REQ-005 · 2 | Request for a nonexistent identifier records no account-specific reset link. | TASK-003, TASK-007 | S, K | **Specified** synthetic recovery and nondisclosure; no unknown-account issuance and separate sink assertion **proposed**. |
| REQ-006 · 1 | After an unexpired link completes a reset, resubmitting that link cannot complete another. | TASK-005, TASK-006, TASK-008 | S, K | **Specified** single use; guarded completion, unusable-link state, sequential/concurrent replay checks **proposed**. |
| REQ-007 · 1 | Link issued at time `t` remains eligible strictly before `t + 30 minutes` if otherwise valid and unused. | TASK-005, TASK-006, TASK-008 | S, K | **Specified** expiry; strict-before validation, usable-form state, injected-clock check **proposed**. |
| REQ-007 · 2 | At or after `t + 30 minutes`, link cannot complete a reset. | TASK-005, TASK-006, TASK-008 | S, K | **Specified** expiry boundary; rejection and exact-boundary/after-boundary tests **proposed**. |
| REQ-008 · 1 | Valid unused-link completion accepts the new password and rejects the old one. | TASK-005, TASK-006, TASK-008 | S, K | **Story-specified** password reset; credential replacement, success state and verification **proposed**. |
| REQ-009 · 1 | Successful reset leaves none of an account’s multiple previously active sessions authenticated. | TASK-002, TASK-005, TASK-008 | S, K | **Specified** all-session sign-out; synthetic session guard, revocation and independent-client test **proposed**. |
| REQ-010 · 1 | Each successful reset records an owner-addressed success notification in the sink. | TASK-005, TASK-008 | S, K | **Specified** owner notification and prototype sink; success-only record and assertion **proposed**. |
| REQ-011 · 1 | Recovery flow under test uses synthetic rather than real user accounts. | TASK-001, TASK-003, TASK-007 | S | **Story-specified** synthetic boundary; isolated account fixtures and verification **proposed**. |
| REQ-012 · 1 | Generated reset links or successful-reset notifications are inspectable in memory without external delivery. | TASK-001, TASK-003, TASK-005, TASK-007, TASK-008 | S | **Story-specified** sink; private injected sink and tests of both record types **proposed**. |
| REQ-013 · 1 | With declared dependencies, app and tests execute under Python 3.12. | TASK-001, TASK-007, TASK-008 | S | **Story-specified** runtime; dependency declaration and pytest execution under that interpreter **proposed**. |
| REQ-014 · 1 | Flask handles sign-in and recovery route requests. | TASK-001, TASK-002, TASK-004, TASK-006, TASK-007 | S | **Story-specified** framework; Flask app/routes and request tests **proposed**. |
| REQ-015 · 1 | Sign-in and recovery page content arrives in the server response without client-side rendering. | TASK-002, TASK-004, TASK-006, TASK-007, TASK-008 | S | **Story-specified** server rendering; HTML routes and Flask-client tests **proposed**; recovery screens are not pictured. |
| REQ-016 · 1 | Python 3.12 pytest exercises pictured entry, neutral response, per-account limit, issuance, single use, expiry, replacement, all-session sign-out and owner notice. | TASK-007, TASK-008 | S, W, K | **Story-specified** tests with visual/standard behavior as inputs; test suite and execution **proposed**, not reported as passing. |
| REQ-017 · 1 | Documentation identifies synthetic accounts and in-memory sink as unsuitable substitutes for production account and delivery systems. | TASK-009 | S | **Story-specified** documentation; limitations text and review **proposed**. |
| REQ-018 · 1 | Inspection finds no deployment performed and no deployment workflow added. | TASK-009 | S | **Story-specified** non-deployment; read-only scope inspection **proposed**, not a claim that future delivery has been inspected. |

## Coverage and evidence limits

- **23/23 acceptance criteria** mapped (REQ-001 has 3; REQ-004, REQ-005 and REQ-007 have 2 each; the other 14 requirements have 1 each). **18/18 requirements** and **9/9 approved implementation tasks** appear in the map. No criterion is assigned a coverage disposition.
- This map is a trace to proposed work, **not evidence of implementation, test execution, accessibility conformance, or deployment status**. The wireframe shows only sign-in appearance and the recovery affordance; recovery pages, live masking, navigation, synthetic sign-in/session behavior and test results remain unobserved.
- No approved predecessor response provided a standalone stage-output reference locator; no `stage_output` citation is supplied. The curated standard does not prescribe an in-memory sink, Flask, Python, page layouts, or a stronger password policy; those points are traced to the story or treated as solution-design proposals, not attributed to the standard.
