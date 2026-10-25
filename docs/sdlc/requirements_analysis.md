# Requirements analysis

**Stage:** `requirements_analysis`  
**Schema version:** `1.0`

## Source references

Each `S`, `A`, or `K` citation below represents the **exact** corresponding source-reference object, including its locator:

- **S — Pinned user story:** `{"source_type":"user_story","source_id":"user_story","locator":"characters 0-8000"}`
- **A — Sign-in wireframe:** `{"source_type":"attachment","source_id":"23b23ac1-a532-42fc-9973-51a1d4954aa6","locator":"sign-in-wireframe.png"}`
- **K — Curated account recovery standard (v1, ACC-STD-01 through ACC-STD-03):** `{"source_type":"knowledge","source_id":"bcf198d8-a628-401c-8641-b726fcbd00b5","locator":"v1, chunk 0"}`

## Requirements

### REQ-001 — Sign-in page presentation

The application displays a sign-in page matching the supplied wireframe's visible content.

- Given a visitor opens the sign-in page, when the page renders, then a narrow bordered panel on a white background contains the “Sign in” heading, a labeled “Email address” input with “name@example.com” placeholder, and a labeled “Password” input.
- Given the sign-in page is displayed, when a visitor enters a password, then its characters are masked.
- Given a visitor opens the sign-in page, when the page renders, then a blue underlined “Forgot password?” link, a dark wide “Sign in” button, and a gray “No account? Contact your administrator.” footer appear as pictured.

Sources: S, A.

### REQ-002 — Recovery entry point

The sign-in page's “Forgot password?” link starts a password-reset request flow.

- Given the sign-in page is displayed, when a visitor follows “Forgot password?”, then the visitor can submit a reset request for an account identifier.

Sources: S, A, K (ACC-STD-01).

### REQ-003 — Non-enumerating request response

The application responds identically to password-reset requests for existing and nonexistent accounts.

- Given equivalent reset requests for a synthetic account identifier and an unknown identifier, when each is submitted under the same conditions, then the visitor-facing responses are identical and do not disclose which account exists.

Sources: S, K (ACC-STD-01).

### REQ-004 — Request rate limit

The application limits password-reset requests to five per account per hour.

- Given five reset requests for the same synthetic account within one hour, when a sixth request is submitted within that hour, then no sixth reset link is issued for that account.
- Given that account's applicable one-hour request period has elapsed, when another reset request is submitted, then the prior period's five requests do not prevent a new eligible request.

Sources: S, K (ACC-STD-01).

### REQ-005 — Recovery link issuance

An accepted password-reset request for a synthetic account makes a reset link available through the in-memory notification sink.

- Given a synthetic account and an eligible reset request, when the request is submitted, then the sink records a reset link for that account.
- Given a nonexistent account identifier, when a reset request is submitted, then no account-specific reset link is recorded.

Sources: S, K (ACC-STD-01, ACC-STD-02).

### REQ-006 — Single-use link

A password-reset link can complete at most one password reset.

- Given an unexpired reset link, when a reset is completed with it and the same link is submitted again, then the later submission cannot complete another reset.

Sources: S, K (ACC-STD-02).

### REQ-007 — Link expiration

A password-reset link expires 30 minutes after issue.

- Given an issued link, when a reset is attempted before 30 minutes have elapsed, then the link remains eligible if otherwise valid.
- Given an issued link, when a reset is attempted at or after 30 minutes from issue, then it cannot complete a reset.

Sources: S, K (ACC-STD-02).

### REQ-008 — Password replacement

Completing a valid password reset replaces the account's password with the submitted new password.

- Given a synthetic account and a valid unused link, when a new password is submitted through the reset flow, then the account accepts the new password and no longer accepts the previous password.

Sources: S, K (ACC-STD-02).

### REQ-009 — All-session sign-out

Completing a password reset signs the account out of every active session.

- Given a synthetic account with multiple active sessions, when its password is successfully reset, then none of those sessions remains authenticated for that account.

Sources: S, K (ACC-STD-02).

### REQ-010 — Owner notification

Every successful password reset produces a notification addressed to the account owner in the in-memory notification sink.

- Given a synthetic account, when its password is successfully reset, then the sink records an owner-addressed successful-reset notification for that reset.

Sources: S, K (ACC-STD-03).

### REQ-011 — Synthetic accounts

The prototype performs account-recovery behavior using synthetic accounts.

- Given the application is configured for testing, when the recovery flow is exercised, then the participating accounts are synthetic rather than real user accounts.

Sources: S.

### REQ-012 — In-memory notification sink

The prototype records recovery notifications in an in-memory sink rather than delivering them through an external service.

- Given a recovery link or successful-reset notification is generated, when its delivery is inspected, then a record is available in the in-memory sink without requiring an external delivery service.

Sources: S.

### REQ-013 — Python runtime

The application runs under Python 3.12.

- Given a Python 3.12 environment with the declared dependencies installed, when the application and tests are started, then they execute under that interpreter.

Sources: S.

### REQ-014 — Flask application

The password-reset application uses Flask to handle its web requests.

- Given the application is running, when a visitor requests its sign-in and recovery routes, then Flask handles those requests.

Sources: S.

### REQ-015 — Server-rendered pages

The application's sign-in and recovery pages are rendered on the server.

- Given a visitor requests a sign-in or recovery page, when the response is received, then its page content is present in the server response without requiring client-side rendering.

Sources: S.

### REQ-016 — Automated pytest coverage

The project provides automated pytest tests for the password-reset application's required behavior.

- Given Python 3.12 and the declared test dependencies, when pytest runs, then automated tests exercise the pictured sign-in entry point, non-enumerating response, per-account request limit, link issuance, single use, expiration, password replacement, all-session sign-out, and owner notification.

Sources: S, A, K (ACC-STD-01 through ACC-STD-03).

### REQ-017 — Production limitations

The project documents the prototype's production limitations.

- Given a reader opens the project documentation, when the production-limitations section is read, then it identifies synthetic accounts and the in-memory notification sink as unsuitable substitutes for production account and delivery systems.

Sources: S.

### REQ-018 — No deployment

The project remains undeployed and does not include a deployment workflow.

- Given the delivered project, when its changes and workflow files are inspected, then no deployment action was performed and no deployment workflow was added.

Sources: S.

## Assumptions

- acceptance-20261025121757-621df1-skill-sentinel — the acceptance requirements skill produced this output.
- The image guides only the visible sign-in page; it does not prescribe sign-in validation or the design of recovery pages.
- An in-memory notification-sink record makes a recovery link inspectable for testing and records post-reset owner notifications; no external delivery channel is assumed.
- Synthetic accounts and ephemeral state are appropriate for this prototype, not production; no persistent account store or delivery provider is specified.
- Tests may establish multiple active sessions for a synthetic account to verify all-session invalidation; no particular session mechanism is assumed.
- An account identifier in the reset request is assumed to be the email address shown on the sign-in page; no extra account-identification or password policy is assumed.
- The complete pinned story contains no end marker, and the image supplies no attachment acceptance marker.

## Open questions (all non-blocking)

- **What sign-in behavior beyond the pictured form is expected?** Working assumption: Only the visible sign-in page is specified; tests may establish sessions to verify the explicitly required sign-out behavior.
- **Are additional notification or reset-link presentation requirements expected?** Working assumption: The in-memory sink contains inspectable reset links and owner notifications, without external delivery or additional UI specifications.
- **Should a later stage specify a persistent store or delivery provider?** Working assumption: Neither is required for this synthetic, in-memory prototype; document the production limitation instead.
- **Is a particular session mechanism required?** Working assumption: Any testable implementation that invalidates all of an account's active sessions meets the standard.
- **Are additional account identifiers or password rules expected?** Working assumption: Use the pictured email-address identifier and do not add an unsupported password policy.
