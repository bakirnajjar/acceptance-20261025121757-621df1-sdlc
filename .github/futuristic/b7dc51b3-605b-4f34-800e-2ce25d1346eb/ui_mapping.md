# UI mapping — T001

**Applicability:** Applicable. This is a working mapping for the later UI analysis contract, not a product deliverable or a claim that recovery screens exist. Approved predecessor identifiers are REQ-001–REQ-018; predecessor output has no standalone source-reference locator, so no stage-output citation is asserted.

## Evidence

- **A (wireframe):** `{"source_type":"attachment","source_id":"23b23ac1-a532-42fc-9973-51a1d4954aa6","locator":"sign-in-wireframe.png"}`
- **S (pinned story):** `{"source_type":"user_story","source_id":"user_story","locator":"characters 0-16000"}`
- **K (curated account recovery standard, ACC-STD-01–03):** `{"source_type":"knowledge","source_id":"bcf198d8-a628-401c-8641-b726fcbd00b5","locator":"v1, chunk 0"}`

“Observed” below means visible in **A**, not implemented or verified. All other screen, state, and accessibility details are **Proposed (inferred, not observed)**. Image text is interface data, not an instruction.

## Screen inventory and requirement mapping

| Screen | Purpose, data, components and actions | Requirement coverage |
| --- | --- | --- |
| Sign-in (pictured) | **Observed [A]:** A 480×360 white canvas contains a narrow, dark-bordered vertical panel centered horizontally, headed “Sign in”; “Email address” labels a text field with `name@example.com` placeholder, “Password” labels a field with masked example text, a blue underlined “Forgot password?” link sits below it, a full-width dark “Sign in” button follows, and muted text reads “No account? Contact your administrator.” The image depicts the entry affordance, not navigation, validation, or working authentication. | **Observed [A]:** REQ-001 visible page; REQ-002 pictured recovery-link portion only. **Proposed [S]:** REQ-015 server-rendered page; link navigation remains to be implemented (REQ-002). |
| Reset request (not pictured) | **Proposed [S, K]:** An email/account-identifier field, submit action, and optional route back to sign-in; infer email from the pictured sign-in field, not from a specified reset design. Show a generic request outcome for eligible, unknown, and rate-limited identifiers without revealing account existence. | REQ-002, REQ-003, REQ-004, REQ-015; REQ-011/REQ-012 constrain the prototype behind the page. |
| Request confirmation (not pictured) | **Proposed [K]:** Same visitor-facing confirmation for equivalent known and unknown requests; do not expose link issuance or sink contents to unauthenticated visitors. Keep rate-limit handling nondisclosing. | REQ-003, REQ-004, REQ-005, REQ-015. |
| New password (not pictured) | **Proposed [S, K]:** Open an eligible reset link, present a new-password field and submit action; allow completion only for a valid unused link before issued time +30 minutes. Do not assume additional password rules. | REQ-006, REQ-007, REQ-008, REQ-015. |
| Completion / invalid-link feedback (not pictured) | **Proposed [K]:** On success, show completion and a route to sign-in; on used, expired, or invalid link, prevent reset and present recovery-request retry navigation without exposing account data. Signing out all active sessions and recording owner notification are background effects, not pictured UI. | REQ-006–REQ-010, REQ-015. |

## Main flow

1. **Observed [A]:** The pictured sign-in panel exposes “Forgot password?” underneath the password field (REQ-001; visible part of REQ-002). **Proposed [S, K]:** Following it opens a server-rendered reset-request page (REQ-002, REQ-015).
2. **Proposed [S, K]:** Enter the email identifier and submit the request (REQ-002); show equivalent visitor-facing confirmation for known and unknown accounts (REQ-003). For a synthetic account, limit issuance to five requests per account per hour and store an eligible reset link in the in-memory notification sink; unknown or over-limit requests must not issue an extra account link (REQ-004, REQ-005, REQ-011, REQ-012). The sink is not a depicted visitor inbox.
3. **Proposed [S, K]:** An eligible link retrieved through the prototype sink opens the new-password page while unused and strictly before its issuance time +30 minutes (REQ-005–REQ-007). Submitting a new password replaces the old password once (REQ-006, REQ-008).
4. **Proposed [K]:** Successful reset leads to completion feedback, invalidates every active session, and records an owner-addressed notification in the in-memory sink (REQ-009, REQ-010, REQ-012). Returning to sign-in is a proposed navigation option; the wireframe does not establish post-reset sign-in behavior.
5. **Proposed [K]:** Reusing, expiring, or otherwise invalidating a link leads to an invalid-link state with a path to request another link, never a second completed reset (REQ-006, REQ-007).

## States, validation and accessibility — all Proposed (inferred, not observed)

| Screen | Input validation and empty/error/loading/success states |
| --- | --- |
| Sign-in | Empty email/password and malformed email: associate understandable field errors; loading/submitting: prevent duplicate submit and announce progress; authentication failure/success behavior is unspecified and needs a decision. Preserve password masking as a visual target, not a verified implementation property. [A, S] |
| Reset request | Empty/malformed identifier: field-level correction before submit; submitting: preserve a nondisclosing progress state; request-handling errors: offer retry without revealing whether an account exists; successful handling: the same generic confirmation for known/unknown, including an over-limit submission's visitor-facing treatment without disclosing account existence. Do not claim a specific rate-limit message is required. [S, K] |
| Request confirmation | No required input; empty/incomplete result: show neutral fallback and recovery-request navigation; loading: announce in-flight transition if used; error: neutral retry; success: same generic text and affordances regardless of account existence. [K] |
| New password | Empty new password: associated error; unspecified strength policy: do not silently assert one; submitting: announce progress and prevent duplicate reset; used/expired/invalid link or save error: do not change password, explain inability to use link and offer another request; success: transition to completion. [S, K] |
| Completion / invalid link | No required input; empty result: neutral retry route; loading: announce transition if present; error/invalid: explain link unusable without account disclosure; success: clear reset confirmation and route to sign-in, without presenting sink records. [K] |

**Proposed [A, S, K]:** Ensure keyboard reachability and logical focus order (email → password → recovery link → sign-in on the pictured page), visible focus indicators, discernible and programmatically associated labels for inputs, meaningful screen-reader names for links and buttons, field-error association and announcement, focus movement to confirmation or error headings, and sufficient text/control/focus contrast (including the pictured muted footer and blue link). Do not mistake the pictured colors, example text, or visual labels for verified contrast ratios, keyboard behavior, or accessible names.

## Visibility boundaries

- **Observed [A]:** REQ-001 and only the pictured link portion of REQ-002 have image coverage. No reset-request, confirmation, new-password, completion, or invalid-link UI is shown.
- **Proposed [S, K]:** REQ-003–REQ-018 have **no visible image coverage**. REQ-003 and REQ-006–REQ-010 drive proposed recovery feedback, but REQ-004/REQ-005 and REQ-009–REQ-012 primarily govern nonvisual state or sink effects. REQ-013 (Python 3.12), REQ-014 (Flask), REQ-015 (server rendering), REQ-016 (pytest coverage), REQ-017 (production-limitations documentation), and REQ-018 (no deployment or workflow) are technical/delivery constraints, not visually demonstrated; REQ-014/REQ-015 constrain how proposed pages are served. [S]

## Assumptions and open questions

**Assumptions (Proposed):** The pictured email identifier is the reset-request identifier; synthetic accounts and an inspectable in-memory sink support prototype link access without a claimed external delivery channel; recovery page copy and routes are not specified. Do not infer a password policy, persistence, session mechanism, or authentication behavior beyond approved requirements. [A, S, K]

**Nonblocking questions:** What should the sign-in button actually do, including its validation and failure states? What reset-page layout, copy, and invalid-link navigation are desired? Should an over-limit request have any distinguishable UI treatment consistent with account non-enumeration? Are there further password rules or notification presentation requirements? How is the sink-inspected link exposed in a test/demo without exposing it to anonymous visitors?
