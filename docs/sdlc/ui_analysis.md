# UI analysis — password-reset prototype

**Applicability:** Applicable. The supplied sign-in wireframe depicts a user-facing entry point. **Observed** below means visible in the image, *not implemented or verified*. All recovery screens, transitions, states, and accessibility treatments below are **Proposed (inferred, not observed)**.

## Evidence and boundary

- **[A] Attachment:** `{"source_type":"attachment","source_id":"23b23ac1-a532-42fc-9973-51a1d4954aa6","locator":"sign-in-wireframe.png"}`
- **[S] Pinned story:** `{"source_type":"user_story","source_id":"user_story","locator":"characters 0-16000"}`
- **[K] Curated account recovery standard, ACC-STD-01–03:** `{"source_type":"knowledge","source_id":"bcf198d8-a628-401c-8641-b726fcbd00b5","locator":"v1, chunk 0"}`

The approved requirements establish REQ-001–REQ-018. The prior-stage tool returned their content but no standalone source-reference locator; the identifiers below are links to that approved coverage, **not** fabricated `stage_output` citations. The image provides no evidence of implemented navigation, validation, authentication, backend state, password rules, accessible behavior, or a delivered notification.

## Screens: purpose, data, components, actions

| Screen | Inventory and requirement coverage |
| --- | --- |
| Sign-in, pictured | **Observed [A]:** A 480×360 white canvas has a narrow dark-bordered panel centered horizontally, heading “Sign in”, labeled “Email address” text field with `name@example.com` placeholder, labeled “Password” field with masked sample, blue underlined “Forgot password?” link beneath the password, full-width dark “Sign in” button, and muted “No account? Contact your administrator.” footer. This covers REQ-001 visually and only the *link affordance* of REQ-002. **Proposed [S, K]:** make the link navigate to reset request and serve this page from the server (REQ-002, REQ-015). Button behavior remains undecided. |
| Reset request, not pictured | **Proposed [A, S, K]:** An email account-identifier input, submit button, and return-to-sign-in link. The identifier is inferred from the pictured email field. Send a request without revealing account existence (REQ-002, REQ-003, REQ-015); per-account throttling and sink issuance are behind the page (REQ-004, REQ-005, REQ-011, REQ-012). |
| Request confirmation, not pictured | **Proposed [S, K]:** Neutral confirmation and retry/navigation, identical visitor-facing result for equivalent known and unknown identifiers, without showing link issuance or sink records (REQ-003, REQ-004, REQ-005, REQ-015). |
| New password, not pictured | **Proposed [S, K]:** Eligible reset link leads to a labeled new-password input and submit button; success is possible only while the link is unused and before issue time +30 minutes (REQ-005–REQ-008, REQ-015). No additional password-strength policy is asserted. |
| Reset completion and invalid-link feedback, not pictured | **Proposed [S, K]:** Clear success feedback with proposed return to sign-in, or invalid/used/expired-link explanation with a new-request path. These are separate outcomes, not evidence of implemented behavior (REQ-006–REQ-010, REQ-015). All-session sign-out and owner notification are background effects, not screen contents. |

## Main user flow

1. **Observed [A]:** The pictured sign-in page exposes “Forgot password?” below the password field (REQ-001; visible portion of REQ-002). **Proposed [S, K]:** Following it opens the server-rendered reset-request form (REQ-002, REQ-015).
2. **Proposed [A, S, K]:** Enter an email identifier and submit. Show the same visitor-facing outcome for equivalent known and unknown accounts (REQ-002, REQ-003). Behind that outcome, permit at most five link-issuing requests per synthetic account per hour (REQ-004); an eligible account gets a link recorded in the in-memory sink, while unknown or over-limit requests do not issue an additional account link (REQ-005, REQ-011, REQ-012). The sink is not a pictured or proposed public inbox.
3. **Proposed [S, K]:** An eligible link obtained via the prototype sink opens the new-password form; submitting an unused link strictly before issuance +30 minutes replaces the old password once (REQ-005–REQ-008). At or after the boundary, an expired link is ineligible (REQ-007).
4. **Proposed [S, K]:** Success transitions to completion feedback. Behind the screen, sign the account out of **every** active session and record an owner-addressed notification in the sink (REQ-009, REQ-010, REQ-012). Return to sign-in is a suggested action, not wireframe-established post-reset behavior.
5. **Proposed [S, K]:** Reuse, expiration, or invalidity leads to invalid-link feedback and a new-request route, never a second successful reset (REQ-006, REQ-007).

## Requirement-to-UI coverage

“Not visual” means no image evidence; it does not mean the requirement is out of scope.

| ID | Image evidence | Proposed screen/flow or nonvisual verification boundary |
| --- | --- | --- |
| REQ-001 | **Observed [A]:** Sign-in panel content. | **Proposed [A, S]:** Implement matching server-rendered page and accessible fields. |
| REQ-002 | **Observed [A]:** “Forgot password?” link only. | **Proposed [S, K]:** Link navigates to reset-request form; destination not pictured. |
| REQ-003 | Not visual. | **Proposed [K]:** Identical visitor-facing confirmation for equivalent existing/nonexistent account requests; test response behavior. |
| REQ-004 | Not visual. | **Proposed [K]:** Behind reset-request screen, enforce five requests per account per hour; test issuance limit and elapsed period, not appearance. |
| REQ-005 | Not visual. | **Proposed [S, K]:** Eligible link recorded in in-memory sink, not exposed on visitor page; inspect sink in tests. |
| REQ-006 | Not visual. | **Proposed [K]:** New-password form once; reused link leads to invalid-link feedback; verify single use. |
| REQ-007 | Not visual. | **Proposed [K]:** Invalid-link state at or after issue time +30 minutes; test boundary. |
| REQ-008 | Not visual. | **Proposed [S, K]:** Successful new-password submission and completion feedback; verify password replacement, not just a success message. |
| REQ-009 | Not visual. | **Proposed [K]:** Background all-session sign-out on success; verify across active sessions. |
| REQ-010 | Not visual. | **Proposed [K]:** Background owner-addressed success notification; inspect sink, not completion copy. |
| REQ-011 | Not visual. | **Proposed [S]:** Use synthetic accounts for prototype flow; verify configuration/data. |
| REQ-012 | Not visual. | **Proposed [S]:** Use in-memory notification sink, not external delivery; verify records. |
| REQ-013 | Not visual. | **Proposed [S]:** Python 3.12 runtime; verify application and tests under it. |
| REQ-014 | Not visual. | **Proposed [S]:** Flask handles sign-in and recovery routes; verify routing, not screenshot. |
| REQ-015 | Not visual as a rendering mechanism. | **Proposed [S]:** Server-render sign-in and all recovery pages; verify response content without client rendering. |
| REQ-016 | Not visual. | **Proposed [S]:** pytest exercises reset behavior, entry point, and edge cases; inspect/run tests. |
| REQ-017 | Not visual. | **Proposed [S]:** Document production limits of synthetic accounts and memory-only sink; inspect documentation. |
| REQ-018 | Not visual. | **Proposed [S]:** No deployment and no deployment workflow; verify project changes and activity separately. |

Only REQ-001 and the pictured portion of REQ-002 have visual evidence. REQ-003–REQ-018 have **no visible image coverage**. In particular, REQ-004/REQ-005 and REQ-009–REQ-012 concern enforcement, state, or delivery; REQ-013–REQ-018 primarily concern technology, tests, documentation, and deployment scope. REQ-014/REQ-015 constrain delivery of the proposed screens but are not verified by their visual appearance.

## Per-screen validation, empty/error/loading states — all Proposed (inferred, not observed)

| Screen | States and validation proposal |
| --- | --- |
| Sign-in | Empty email or password and malformed email: associated field errors; loading: announce submitting and prevent duplicate submit; failure: intelligible error without claiming a specified authentication mechanism; success: behavior requires decision. Preserve the pictured password masking as a visual target, not proof of implementation. [A, S] |
| Reset request | Empty/malformed identifier: field-level correction before processing; loading: neutral progress without account-status leak; error: nondisclosing retry; success: equivalent generic response for known and unknown accounts. Over-limit feedback must not disclose account existence; specific wording is undecided. [A, S, K] |
| Request confirmation | No required input; empty/missing result: neutral fallback and request-again route; loading if transition exists: announce it; error: neutral retry; success: identical copy and actions for equivalent known/unknown requests. [S, K] |
| New password | Empty new password: associated error; further password policy unspecified; loading: announce progress and prevent duplicate reset; error, used/expired/invalid link, or processing failure: no password change, explanatory retry path; success: completion. [S, K] |
| Completion and invalid-link feedback | No required input; empty result: neutral fallback route; loading if transition exists: announce it; error/invalid: explain unusable link without account data, offer new request; success: explicit completed-reset confirmation and proposed sign-in route, no sink disclosure. [S, K] |

## Accessibility — Proposed (inferred, not observed) [A, S, K]

Ensure keyboard access to every field, button, and link with a logical focus order (on the pictured panel: email → password → recovery link → sign-in), visible focus indicators, programmatically associated input labels and discernible screen-reader names for actions. Associate field errors with inputs; announce errors, submitting status, and confirmation changes; manage focus to new confirmation or error headings. Check text, control, and focus-indicator contrast, especially the pictured blue link and muted footer. Pictured visual labels, colors, and masking do **not** verify actual keyboard behavior, accessible names, contrast ratios, or password-input semantics.

## Assumptions and nonblocking questions

**Assumptions:** The pictured email field supplies the proposed recovery identifier [A, S]; recovery layouts, copy, routes, and UI states are not specified [A, S, K]. Synthetic accounts and an inspectable in-memory sink are prototype mechanisms, not a claim of external delivery [S]; no particular session implementation, persistent store, password policy, or sign-in outcome is implied. The approved predecessor IDs can be linked, but a missing prior-output locator cannot be cited.

1. What should the pictured Sign in button do, including authentication failure behavior?
2. What recovery-screen layout, wording, and invalid-link/new-request navigation are desired?
3. Should over-limit requests have distinguishable feedback, and how would it preserve account non-enumeration?
4. Are additional password rules or notification presentation requirements intended?
5. How should testers or a demo inspect sink-held links without exposing them to unauthenticated visitors?

These decisions do not block this UI mapping. No implementation, test execution, external delivery, or deployment is claimed.
