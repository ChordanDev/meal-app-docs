# Apply Progress: implement-auth-account-mvp

## 2026-06-07 — Work Unit 1: Account graph schema and access calculation

### Structured status consumed

- `changeName`: `implement-auth-account-mvp`
- `phase`: `apply`
- `applyState`: `ready`
- `artifactStore`: `openspec`
- `deliveryPath`: `auto-chain`
- `assignedWorkUnit`: `Work Unit 1 — Account graph schema and access calculation (PR 1)`
- `actionContext.mode`: `implementation`
- `allowedEditRoots`:
  - `/Users/luccagiordana/Documents/proyectoApp/my_food_back`
  - `/Users/luccagiordana/Documents/proyectoApp/meal-app-docs/openspec/changes/implement-auth-account-mvp`
- Action context warnings: none.

### Workload / PR boundary

- Implemented only PR 1 / Work Unit 1.
- Did not implement email codes, sessions, controllers, routes, `/me`, frontend integration, or Work Units 2–4.
- Review forecast remains high for the full slice; chained PR strategy remains `stacked-to-main`.

### Completed tasks and persisted checkbox updates

Updated `openspec/changes/implement-auth-account-mvp/tasks.md` from `- [ ]` to `- [x]` for:

- Pre-implementation backend working tree check.
- Work Unit 1 RED context tests.
- Work Unit 1 fixture-helper decision: no fixture helper was needed because tests use `DataCase` directly.
- Work Unit 1 migration generation for users/accounts/memberships.
- Work Unit 1 schemas for User, Account, Membership.
- Work Unit 1 `Accounts` context functions.
- Work Unit 1 deterministic time seam via explicit `now` arguments.
- Work Unit 1 focused verification command.
- Work Unit 1 acceptance mapping and rollback-boundary notes.

### Files changed

Backend (`../my_food_back`):

- `lib/my_food_back/accounts.ex`
- `lib/my_food_back/accounts/user.ex`
- `lib/my_food_back/accounts/account.ex`
- `lib/my_food_back/accounts/membership.ex`
- `priv/repo/migrations/20260607201017_create_account_graph.exs`
- `test/my_food_back/accounts_test.exs`

OpenSpec:

- `openspec/changes/implement-auth-account-mvp/tasks.md`
- `openspec/changes/implement-auth-account-mvp/apply-progress.md`

### TDD Cycle Evidence

| Cycle | Phase | Command | Result | Evidence |
|-------|-------|---------|--------|----------|
| WU1 | RED | `cd my_food_back && mix test test/my_food_back/accounts_test.exs` | Failed as expected | Compile failure: `MyFoodBack.Accounts.Account.__struct__/1 is undefined`, proving tests were written before schemas/context. |
| WU1 | GREEN | `cd my_food_back && mix test test/my_food_back/accounts_test.exs` | Passed | `5 tests, 0 failures`. |
| WU1 | REFACTOR / VERIFY | `cd my_food_back && mix format && mix test test/my_food_back/accounts_test.exs` | Passed | Formatting completed and focused tests passed: `5 tests, 0 failures`. |
| WU1 | BROADER VERIFY | `cd my_food_back && mix test` | Passed | Full current backend suite passed: `7 tests, 0 failures`. |

### Acceptance mapping

- Signup account graph prerequisite: covered by `create_individual_account/2` tests asserting exactly one normalized User, Individual Account, owner Membership, 10-day trial, and default onboarding fields.
- Trial Gate Access Semantics: covered by `access_state/2` tests for `now < trial_ends_at`, exact boundary lock, after-boundary lock, and active subscription override.
- Current account loading prerequisite for future `/me`: covered by `get_current_account/1` test returning active owner Membership and Individual Account.

### Deviations from design

- No separate `Clock` module was added. Work Unit 1 uses explicit `now` arguments for deterministic tests, which was an allowed design/task option.
- No fixture module was added because Work Unit 1 tests are concise and use `DataCase` directly.
- UUID primary keys were used for the new account graph tables because the design recommended UUIDs when no project standard was fixed.

### Remaining tasks

Unchecked persisted tasks remain:

```text
- [ ] Add failing tests in `../my_food_back/test/my_food_back/auth/email_code_test.exs` for signup/login request rules: hashed 6-digit code only, no User on request, duplicate signup `email_already_exists`, unknown login `email_not_found`, 10-minute expiry, 5-attempt cap, new code invalidation, 60-second cooldown, and stable `rate_limited` result.
- [ ] Add Swoosh/test delivery assertions or adapter fakes in `../my_food_back/test/support/` without exposing plaintext code in DB assertions.
- [ ] Generate migrations for `email_codes` and persisted rate-limit events (or the minimal persisted structures chosen for email/IP/device practical limits) under `../my_food_back/priv/repo/migrations/`.
- [ ] Implement `../my_food_back/lib/my_food_back/auth/email_code.ex`, core request/verify functions in `../my_food_back/lib/my_food_back/auth.ex`, and stable auth error tuples/codes.
- [ ] Implement `../my_food_back/lib/my_food_back/email_delivery.ex` with a Swoosh-backed adapter boundary using existing `MyFoodBack.Mailer` local/test configuration.
- [ ] Implement `../my_food_back/lib/my_food_back/rate_limits.ex` for cooldown plus email/IP/device practical rate checks, hashing raw identifiers before persistence.
- [ ] Add tests for both `signup` and `login` flows sharing the same security rules without cross-invalidating each other.
- [ ] Verify with `cd ../my_food_back && mix test test/my_food_back/auth/email_code_test.exs`.
- [ ] Acceptance mapping: Explicit Signup Code Request, Explicit Login Code Request, Email Code Security Rules, and Standard Error Response Contract.
- [ ] Rollback boundary: remove email-code/rate-limit migrations and context code; PR 1 account graph remains usable independently.
- [ ] Add failing tests in `../my_food_back/test/my_food_back/auth/session_test.exs` for device-scoped session creation, second-device login preserving first session, valid refresh rotating refresh token, old/revoked refresh rejection, and logout revoking only current session.
- [ ] Generate `sessions` migration under `../my_food_back/priv/repo/migrations/` with hashed refresh token, device/IP/user-agent metadata, expiry, and revocation fields.
- [ ] Implement `../my_food_back/lib/my_food_back/auth/session.ex` and `../my_food_back/lib/my_food_back/auth/tokens.ex` using short-lived signed access tokens and high-entropy refresh secrets stored only as hashes.
- [ ] Extend `../my_food_back/lib/my_food_back/auth.ex` so signup verification transaction consumes code, creates account graph, creates session, and returns tokens; login verification consumes code and creates only a new session.
- [ ] Implement refresh-token rotation and current-session logout semantics.
- [ ] Add locked-account login test proving authentication still succeeds and returned `me.account.access.canUseApp` can be false after trial expiry.
- [ ] Verify with `cd ../my_food_back && mix test test/my_food_back/auth/session_test.exs test/my_food_back/auth/email_code_test.exs`.
- [ ] Acceptance mapping: Signup Code Verification Creates User Account And Session, Login Code Verification Creates Device Session, Session Refresh And Logout, Locked account can still authenticate.
- [ ] Rollback boundary: disable session-dependent APIs and revoke/delete non-production sessions without deleting users/accounts.
- [ ] Add failing controller tests in `../my_food_back/test/my_food_back_web/controllers/auth_controller_test.exs` for all auth endpoints under `/api/auth/*`, exact success response shapes, and `error.code` envelopes.
- [ ] Add failing controller tests in `../my_food_back/test/my_food_back_web/controllers/me_controller_test.exs` for authenticated `/api/me`, unauthenticated rejection, omitted full preferences, active trial access, expired trial lock, and active subscription override.
- [ ] Update `../my_food_back/lib/my_food_back_web/router.ex` with `/api/auth/signup/request-code`, `/api/auth/signup/verify-code`, `/api/auth/login/request-code`, `/api/auth/login/verify-code`, `/api/auth/refresh`, authenticated `/api/auth/logout`, and authenticated `/api/me`.
- [ ] Implement `../my_food_back/lib/my_food_back_web/controllers/auth_controller.ex` and `auth_json.ex` using Phoenix 1.8 JSON serializers, not `Phoenix.View`.
- [ ] Implement `../my_food_back/lib/my_food_back_web/controllers/me_controller.ex` and `me_json.ex` with the spec response shape and no full preferences collection.
- [ ] Implement `../my_food_back/lib/my_food_back_web/plugs/authenticate_session.ex` and optional `require_unlocked_account.ex` for future internal app-data routes; do not apply locked-account denial to auth routes or `/me`.
- [ ] Add integration tests proving request-code → verify-code → `/me` works for signup and login, including stable codes: `code_expired`, `code_invalid`, `too_many_attempts`, `rate_limited`, `email_already_exists`, `email_not_found`.
- [ ] Verify with `cd ../my_food_back && mix test`.
- [ ] Run final backend quality gate per `../my_food_back/AGENTS.md`: `cd ../my_food_back && mix precommit`.
- [ ] Acceptance mapping: Current User Contract, Protected App Data Gate backend SHOULD behavior, Standard Error Response Contract, and all endpoint response shapes in `spec.md`.
- [ ] Rollback boundary: remove router entries/controllers/plugs while preserving lower-level contexts and migrations if needed.
- [ ] After backend PRs pass, add a pointer in the relevant frontend integration issue/branch to `meal-app-docs/openspec/changes/implement-auth-account-mvp/specs/auth-account-trial/spec.md`; do not duplicate product decisions into `../my-expo-app`.
- [ ] Confirm frontend consumers use `/api/me` `account.access.canUseApp` as the primary gate and handle `trial_expired`, but leave screen/token-storage implementation to the frontend slice.
- [ ] All spec scenarios in `openspec/changes/implement-auth-account-mvp/specs/auth-account-trial/spec.md` have corresponding automated backend tests or an explicit justified exception.
- [ ] Strict TDD evidence is present in commit history or PR notes for each work unit: RED test first, GREEN implementation, TRIANGULATE edge case, REFACTOR/VERIFY.
- [ ] Full test command passes: `cd ../my_food_back && mix test`.
- [ ] Final precommit passes: `cd ../my_food_back && mix precommit`.
- [ ] Shared docs remain the source of truth; backend/frontend repos contain implementation and pointers only, not copied product decisions.
```

## 2026-06-07 — Work Unit 2: Email code request/verify primitives

### Structured status consumed

- `changeName`: `implement-auth-account-mvp`
- `phase`: `apply`
- `applyState`: `ready`
- `artifactStore`: `openspec`
- `deliveryPath`: `auto-chain`
- `assignedWorkUnit`: `Work Unit 2 — Email code request/verify primitives (PR 2)`
- `actionContext.mode`: `implementation`
- `allowedEditRoots`:
  - `/Users/luccagiordana/Documents/proyectoApp/my_food_back`
  - `/Users/luccagiordana/Documents/proyectoApp/meal-app-docs/openspec/changes/implement-auth-account-mvp`
- Action context warnings: none.

### Workload / PR boundary

- Implemented only Work Unit 2 on backend branch `feat/email-code-auth`.
- Did not implement sessions/tokens/refresh/logout, controllers, routes, `/me`, frontend integration, or Work Units 3–4.
- Backend Work Unit 2 added about 632 new lines across tests, schemas, migrations, auth context, email delivery, and rate limits, exceeding the 400-line review budget. The work remains a single cohesive email-code PR boundary, but should receive careful review.

### Completed tasks and persisted checkbox updates

Updated `openspec/changes/implement-auth-account-mvp/tasks.md` from `- [ ]` to `- [x]` for all Work Unit 2 RED, GREEN, and TRIANGULATE / VERIFY tasks:

- Email-code context tests for signup/login request rules, hashed storage, duplicate signup, missing login email, expiry, attempt cap, invalidation, cooldown, rate limit, and cross-flow isolation.
- Swoosh test delivery assertions without asserting plaintext code from DB storage.
- `email_codes` and `rate_limit_events` migrations.
- `Auth.EmailCode`, `Auth`, `EmailDelivery`, and `RateLimits` implementations.
- Focused verification command and Work Unit 2 acceptance/rollback notes.

### Files changed

Backend (`../my_food_back`):

- `lib/my_food_back/auth.ex`
- `lib/my_food_back/auth/email_code.ex`
- `lib/my_food_back/email_delivery.ex`
- `lib/my_food_back/rate_limits.ex`
- `lib/my_food_back/rate_limits/event.ex`
- `priv/repo/migrations/20260607232722_create_email_codes.exs`
- `priv/repo/migrations/20260607232723_create_rate_limit_events.exs`
- `test/my_food_back/auth/email_code_test.exs`

OpenSpec:

- `openspec/changes/implement-auth-account-mvp/tasks.md`
- `openspec/changes/implement-auth-account-mvp/apply-progress.md`

### TDD Cycle Evidence

| Cycle | Phase | Command | Result | Evidence |
|-------|-------|---------|--------|----------|
| WU2 | RED | `cd my_food_back && mix test test/my_food_back/auth/email_code_test.exs` | Failed as expected | Compile failure: `MyFoodBack.Auth.EmailCode.__struct__/1 is undefined`, proving tests were written before auth/email-code implementation. |
| WU2 | GREEN | `cd my_food_back && mix test test/my_food_back/auth/email_code_test.exs` | Passed | Focused email-code suite passed: `10 tests, 0 failures`. |
| WU2 | REFACTOR / VERIFY | `cd my_food_back && mix format && mix test test/my_food_back/auth/email_code_test.exs && mix test` | Passed | Formatting completed, focused tests passed `10 tests, 0 failures`, full backend suite passed `19 tests, 0 failures`. |
| WU2 | BROADER VERIFY | `cd my_food_back && mix precommit` | Passed | Final backend precommit passed with `19 tests, 0 failures`. |

### Acceptance mapping

- Explicit Signup Code Request: covered by `request_signup_code/2` tests for normalized email, code delivery, hashed DB storage only, no User creation on request, duplicate signup `email_already_exists`, cooldown, and rate limiting.
- Explicit Login Code Request: covered by `request_login_code/2` tests for existing-user requirement and `email_not_found` for unknown login email.
- Email Code Security Rules: covered by tests for 6-digit delivered code, 10-minute expiry boundary at `expires_at`, max 5 failed attempts with sixth request returning `too_many_attempts`, new-code invalidation, per-flow isolation, and hashed IP/device identifiers.
- Standard Error Response Contract: lower-level context functions return stable `{:error, %{code: "..."}}` tuples for `email_already_exists`, `email_not_found`, `rate_limited`, `code_expired`, `code_invalid`, and `too_many_attempts`. HTTP error envelopes remain for Work Unit 4 controllers.

### Deviations from design

- No separate fake email adapter module was added; tests use the configured Swoosh test adapter and `Swoosh.TestAssertions`.
- Rate limiting is persisted for request-code events by email/IP/device plus required per-code verification attempt caps. Additional verify-code rolling IP/device limits can be hardened later if needed, but Work Unit 2 covers the required stable `rate_limited` request behavior and per-code brute-force cap.
- Signup/login verification currently consumes valid codes and returns the consumed `EmailCode`. Session/account creation remains intentionally deferred to Work Unit 3.

### Remaining tasks

Unchecked persisted tasks remain:

```text
- [ ] Add failing tests in `../my_food_back/test/my_food_back/auth/session_test.exs` for device-scoped session creation, second-device login preserving first session, valid refresh rotating refresh token, old/revoked refresh rejection, and logout revoking only current session.
- [ ] Generate `sessions` migration under `../my_food_back/priv/repo/migrations/` with hashed refresh token, device/IP/user-agent metadata, expiry, and revocation fields.
- [ ] Implement `../my_food_back/lib/my_food_back/auth/session.ex` and `../my_food_back/lib/my_food_back/auth/tokens.ex` using short-lived signed access tokens and high-entropy refresh secrets stored only as hashes.
- [ ] Extend `../my_food_back/lib/my_food_back/auth.ex` so signup verification transaction consumes code, creates account graph, creates session, and returns tokens; login verification consumes code and creates only a new session.
- [ ] Implement refresh-token rotation and current-session logout semantics.
- [ ] Add locked-account login test proving authentication still succeeds and returned `me.account.access.canUseApp` can be false after trial expiry.
- [ ] Verify with `cd ../my_food_back && mix test test/my_food_back/auth/session_test.exs test/my_food_back/auth/email_code_test.exs`.
- [ ] Acceptance mapping: Signup Code Verification Creates User Account And Session, Login Code Verification Creates Device Session, Session Refresh And Logout, Locked account can still authenticate.
- [ ] Rollback boundary: disable session-dependent APIs and revoke/delete non-production sessions without deleting users/accounts.
- [ ] Add failing controller tests in `../my_food_back/test/my_food_back_web/controllers/auth_controller_test.exs` for all auth endpoints under `/api/auth/*`, exact success response shapes, and `error.code` envelopes.
- [ ] Add failing controller tests in `../my_food_back/test/my_food_back_web/controllers/me_controller_test.exs` for authenticated `/api/me`, unauthenticated rejection, omitted full preferences, active trial access, expired trial lock, and active subscription override.
- [ ] Update `../my_food_back/lib/my_food_back_web/router.ex` with `/api/auth/signup/request-code`, `/api/auth/signup/verify-code`, `/api/auth/login/request-code`, `/api/auth/login/verify-code`, `/api/auth/refresh`, authenticated `/api/auth/logout`, and authenticated `/api/me`.
- [ ] Implement `../my_food_back/lib/my_food_back_web/controllers/auth_controller.ex` and `auth_json.ex` using Phoenix 1.8 JSON serializers, not `Phoenix.View`.
- [ ] Implement `../my_food_back/lib/my_food_back_web/controllers/me_controller.ex` and `me_json.ex` with the spec response shape and no full preferences collection.
- [ ] Implement `../my_food_back/lib/my_food_back_web/plugs/authenticate_session.ex` and optional `require_unlocked_account.ex` for future internal app-data routes; do not apply locked-account denial to auth routes or `/me`.
- [ ] Add integration tests proving request-code → verify-code → `/me` works for signup and login, including stable codes: `code_expired`, `code_invalid`, `too_many_attempts`, `rate_limited`, `email_already_exists`, `email_not_found`.
- [ ] Verify with `cd ../my_food_back && mix test`.
- [ ] Run final backend quality gate per `../my_food_back/AGENTS.md`: `cd ../my_food_back && mix precommit`.
- [ ] Acceptance mapping: Current User Contract, Protected App Data Gate backend SHOULD behavior, Standard Error Response Contract, and all endpoint response shapes in `spec.md`.
- [ ] Rollback boundary: remove router entries/controllers/plugs while preserving lower-level contexts and migrations if needed.
- [ ] After backend PRs pass, add a pointer in the relevant frontend integration issue/branch to `meal-app-docs/openspec/changes/implement-auth-account-mvp/specs/auth-account-trial/spec.md`; do not duplicate product decisions into `../my-expo-app`.
- [ ] Confirm frontend consumers use `/api/me` `account.access.canUseApp` as the primary gate and handle `trial_expired`, but leave screen/token-storage implementation to the frontend slice.
- [ ] All spec scenarios in `openspec/changes/implement-auth-account-mvp/specs/auth-account-trial/spec.md` have corresponding automated backend tests or an explicit justified exception.
- [ ] Strict TDD evidence is present in commit history or PR notes for each work unit: RED test first, GREEN implementation, TRIANGULATE edge case, REFACTOR/VERIFY.
- [ ] Full test command passes: `cd ../my_food_back && mix test`.
- [ ] Final precommit passes: `cd ../my_food_back && mix precommit`.
- [ ] Shared docs remain the source of truth; backend/frontend repos contain implementation and pointers only, not copied product decisions.
```

## 2026-06-08 — Work Unit 3: Sessions, tokens, refresh, and logout

### Structured status consumed

- `changeName`: `implement-auth-account-mvp`
- `phase`: `apply`
- `artifactStore`: `openspec`
- `deliveryPath`: `auto-chain`
- `assignedWorkUnit`: `Work Unit 3 — Sessions & Tokens`
- `actionContext.allowedEditRoots`:
  - `/Users/luccagiordana/Documents/proyectoApp/my_food_back`
  - `/Users/luccagiordana/Documents/proyectoApp/meal-app-docs/openspec/changes/implement-auth-account-mvp`
- Action context warnings: none.

### Workload / PR boundary

- Implemented only Work Unit 3 on backend branch `feat/session-token-auth`.
- Did not implement controllers, routes, `/me` HTTP endpoint, frontend integration, or Work Unit 4.
- Backend diff is above the 400-line review budget but remains a cohesive session/token primitive PR boundary.

### Completed tasks and persisted checkbox updates

Updated `openspec/changes/implement-auth-account-mvp/tasks.md` from `- [ ]` to `- [x]` for all Work Unit 3 RED, GREEN, and TRIANGULATE / VERIFY tasks:

- Session tests for device-scoped creation, second-device preservation, refresh rotation, old/revoked/expired refresh rejection, and logout current-session revocation.
- `sessions` migration with hashed refresh token, device/IP/user-agent metadata, expiry, rotation, and revocation fields.
- `Auth.Session` and `Auth.Tokens` implementations.
- `Auth` verification flows now create account/session material, login creates sessions without new accounts, refresh rotates, and logout revokes one session.
- Locked-account login test and Work Unit 3 verification/acceptance/rollback notes.

### Files changed

Backend (`../my_food_back`):

- `lib/my_food_back/accounts.ex`
- `lib/my_food_back/auth.ex`
- `lib/my_food_back/auth/session.ex`
- `lib/my_food_back/auth/tokens.ex`
- `priv/repo/migrations/20260608000144_create_sessions.exs`
- `test/my_food_back/auth/email_code_test.exs`
- `test/my_food_back/auth/session_test.exs`

OpenSpec:

- `openspec/changes/implement-auth-account-mvp/tasks.md`
- `openspec/changes/implement-auth-account-mvp/apply-progress.md`

### TDD Cycle Evidence

| Cycle | Phase | Command | Result | Evidence |
|-------|-------|---------|--------|----------|
| WU3 | RED | `cd my_food_back && mix test test/my_food_back/auth/session_test.exs` | Failed as expected | Session tests failed because `verify_*_code` still returned `EmailCode` structs and `Auth.refresh_session/2` / `Auth.logout/2` were undefined. |
| WU3 | GREEN | `cd my_food_back && mix test test/my_food_back/auth/session_test.exs` | Passed after implementation | Session primitive suite passed after adding sessions migration/schema, token helpers, verification session creation, refresh rotation, replay/revoked/expired rejection, and logout. |
| WU3 | TRIANGULATE / VERIFY | `cd my_food_back && mix test test/my_food_back/auth/session_test.exs test/my_food_back/auth/email_code_test.exs` | Passed | Focused auth suites passed: `16 tests, 0 failures`. |
| WU3 | BROADER VERIFY | `cd my_food_back && mix test` | Passed | Full backend suite passed: `25 tests, 0 failures`. |
| WU3 | REFACTOR / PRECOMMIT | `cd my_food_back && mix format && mix precommit` | Passed | Formatting completed and precommit passed: `25 tests, 0 failures`. |

### Acceptance mapping

- Signup Code Verification Creates User Account And Session: covered by session test asserting signup verification creates User/Account/Membership/session, returns access/refresh tokens, and stores only hashed refresh token.
- Login Code Verification Creates Device Session: covered by second-device login test asserting two active sessions and no first-session revocation.
- Session Refresh And Logout: covered by refresh rotation/replay rejection, revoked/expired refresh rejection, and logout-only-current-session tests.
- Locked account can still authenticate: covered by expired-trial login test returning `me.account.access.can_use_app: false` with `trial_expired` while still issuing session material.

### Deviations from design

- Refresh token rotation creates a new session row linked by `rotated_from_id` and revokes the old row with `revoked_reason: "rotated"`, rather than replacing the token hash in place. This preserves rotation history and enables replay detection.
- Access tokens use `Phoenix.Token` with signed payload containing `session_id`, `user_id`, and `exp`; no plaintext access token is stored.
- Lower-level auth responses use snake_case maps. HTTP/camelCase envelopes remain for Work Unit 4 controllers/serializers.
- `Accounts.create_individual_account_multi/2` was added so signup verification can compose account graph creation inside the auth transaction.

### Remaining tasks

Unchecked persisted tasks remain:

```text
- [ ] Add failing controller tests in `../my_food_back/test/my_food_back_web/controllers/auth_controller_test.exs` for all auth endpoints under `/api/auth/*`, exact success response shapes, and `error.code` envelopes.
- [ ] Add failing controller tests in `../my_food_back/test/my_food_back_web/controllers/me_controller_test.exs` for authenticated `/api/me`, unauthenticated rejection, omitted full preferences, active trial access, expired trial lock, and active subscription override.
- [ ] Update `../my_food_back/lib/my_food_back_web/router.ex` with `/api/auth/signup/request-code`, `/api/auth/signup/verify-code`, `/api/auth/login/request-code`, `/api/auth/login/verify-code`, `/api/auth/refresh`, authenticated `/api/auth/logout`, and authenticated `/api/me`.
- [ ] Implement `../my_food_back/lib/my_food_back_web/controllers/auth_controller.ex` and `auth_json.ex` using Phoenix 1.8 JSON serializers, not `Phoenix.View`.
- [ ] Implement `../my_food_back/lib/my_food_back_web/controllers/me_controller.ex` and `me_json.ex` with the spec response shape and no full preferences collection.
- [ ] Implement `../my_food_back/lib/my_food_back_web/plugs/authenticate_session.ex` and optional `require_unlocked_account.ex` for future internal app-data routes; do not apply locked-account denial to auth routes or `/me`.
- [ ] Add integration tests proving request-code → verify-code → `/me` works for signup and login, including stable codes: `code_expired`, `code_invalid`, `too_many_attempts`, `rate_limited`, `email_already_exists`, `email_not_found`.
- [ ] Verify with `cd ../my_food_back && mix test`.
- [ ] Run final backend quality gate per `../my_food_back/AGENTS.md`: `cd ../my_food_back && mix precommit`.
- [ ] Acceptance mapping: Current User Contract, Protected App Data Gate backend SHOULD behavior, Standard Error Response Contract, and all endpoint response shapes in `spec.md`.
- [ ] Rollback boundary: remove router entries/controllers/plugs while preserving lower-level contexts and migrations if needed.
- [ ] After backend PRs pass, add a pointer in the relevant frontend integration issue/branch to `meal-app-docs/openspec/changes/implement-auth-account-mvp/specs/auth-account-trial/spec.md`; do not duplicate product decisions into `../my-expo-app`.
- [ ] Confirm frontend consumers use `/api/me` `account.access.canUseApp` as the primary gate and handle `trial_expired`, but leave screen/token-storage implementation to the frontend slice.
- [ ] All spec scenarios in `openspec/changes/implement-auth-account-mvp/specs/auth-account-trial/spec.md` have corresponding automated backend tests or an explicit justified exception.
- [ ] Strict TDD evidence is present in commit history or PR notes for each work unit: RED test first, GREEN implementation, TRIANGULATE edge case, REFACTOR/VERIFY.
- [ ] Full test command passes: `cd ../my_food_back && mix test`.
- [ ] Final precommit passes: `cd ../my_food_back && mix precommit`.
- [ ] Shared docs remain the source of truth; backend/frontend repos contain implementation and pointers only, not copied product decisions.
```
