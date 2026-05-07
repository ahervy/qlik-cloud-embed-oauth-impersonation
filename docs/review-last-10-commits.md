# Review: Last 10 Commits

Scope: `HEAD~10..HEAD` (commits `68ab8eb` through `23636eb`)

## Summary

The change set is directionally solid: it adds the `/user-attributes` QIX endpoint, improves the raw data panel, tightens static file serving, makes tenant URI setup easier, and expands Playwright coverage around backend health and user-facing flows.

Because this repository is used as external documentation, the main risks are clarity and diagnostic accuracy. I kept app behavior unchanged and made local edits only where wording or test reporting could mislead users.

## Findings

1. The data panel described the backend QIX fetch as a server-side REST call and claimed embedded selections update the server engine session. That wording over-promised behavior and mixed transport concepts, so it now says the backend fetches the table through a QIX call and refresh requests the latest data from that server-side session.

2. The README troubleshooting section still said the project uses `csrf-sync`, but the final code and dependencies use `csurf` with `cookie-parser`. The README now matches the implementation.

3. The backend health test used `test.fail(true, msg)` when `/access-token` returned 401. In Playwright, that marks the test as expected to fail, which can hide a real OAuth impersonation failure. It now throws an error so CI reports the failure correctly.

4. The login page hard-coded the `oauth_gen_` prefix in user-facing copy. Since `USER_PREFIX` is configurable and now defaults server-side when unset, the copy now says the configured prefix is used, with `oauth_gen_` as the default.

## Local Changes Made

- `src/home.html`: simplified the data panel explanation.
- `tests/core-functionality.spec.js`: aligned the assertion with the new QIX wording.
- `README.md`: fixed CSRF package wording and added `getUserAttributes` to the helper list.
- `src/login.html`: clarified configurable user prefix copy.
- `tests/backend-health.spec.js`: made frontend token failures fail the test directly.

## Residual Risk

The richer Playwright coverage depends on a live Qlik Cloud tenant and demo app content. That is appropriate for this repo, but tests that assert exact visible Qlik data values can still be sensitive to app changes, tenant latency, or Qlik UI rendering changes.

## Verification

- `git diff --check`: passed.
- `node --check server.js`: passed.
- `npx playwright test --list`: passed, discovering 42 tests across Chromium and WebKit.
- `npx playwright test tests/backend-health.spec.js tests/core-functionality.spec.js --project=chromium`: 7 passed, 4 failed, and 5 did not run. The first failing test is backend user provisioning (`GET /` after login), where Qlik returns `OAUTH-14: Invalid client_id - OAuth client is not authorized`. The UI failures are downstream timeouts because the dashboard is not served after that 500.

## Recommendation

Before publishing the docs-facing update, run the Playwright suite against the intended tenant and confirm the screenshots/docs still match the live demo app.
