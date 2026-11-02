# AGENTS.md

Repository-wide rules for coding agents. Also follow applicable nested
`AGENTS.md` files, especially `androidApp/AGENTS.md`.

## Source of Truth and Scope

- Read relevant source, tests, build configuration, and nested instructions
  before editing. Current code and effective tests take precedence over
  historical migration status in documentation.
- Keep changes scoped to the request. Reuse established components and follow
  existing naming, concurrency, error-handling, and dependency patterns.
- Do not perform unrelated refactoring, reformatting, dependency upgrades, or
  documentation rewrites. Preserve existing user changes.
- Documentation is maintained separately by the repository owner; do not update
  it as a routine part of a code change unless explicitly requested.

## Shared Architecture

`shared/src/commonMain` owns shared business semantics: protocol decoding,
reducers, state machines, projections, history merging/deduplication, questions,
approvals, session control, queues, voice-input rules, and shared file state.

- Put reusable business rules in KMP, not parallel Android and Swift versions.
- Never add a second reducer, projection, or history-merge algorithm for a
  domain already owned by KMP.
- Do not introduce shadow business-state implementations, migration switches,
  or runtime flags that preserve competing sources of truth.
- Keep Compose, SwiftUI, UIKit, and Android framework dependencies out of the
  shared module.

Platform code owns native UI, lifecycle, permissions, secure storage, files,
media, notifications, and native transport integration. Android uses the shared
Gateway runtime; iOS intentionally retains its Swift `GatewayClient` and KMP
adapter. This transport exception is not permission to move business rules into
Swift.

## iOS State Flow

Use `View -> AppStore/Intent -> KMPSharedAdapter -> shared Store`. Shared Stores
publish ordered events containing state patches and platform effect descriptors.

- Swift `@Published` values are rebuildable UI mirrors, never independent
  business-state sources. Do not mutate them from arbitrary business logic.
- Validate event schema, sequence, intent correlation, and patch/effect
  consistency before publishing UI state or performing I/O.
- Preserve fail-closed handling for invalid events and structured errors across
  Kotlin/Native; do not allow undeclared exceptions to cross that boundary.
- Keep adapter/UI publication on the established MainActor boundary. Use
  coarse-grained facades rather than coupling views to reducers or coroutines.
- Preserve incremental streaming and display-link/FIFO batching. Avoid full
  list/JSON copies for every token or unmeasured whole-state rebuilds.

## Platform Requirements

- Maintain Android API 24+ and iOS 16.1+ compatibility. Do not raise deployment
  targets or remove working compatibility paths merely to simplify code.
- Align protocol and business semantics across platforms; native UI and system
  behavior may differ. Explain and validate intentional platform differences
  rather than requiring identical interactions or pixels.
- Preserve lifecycle cancellation, request correlation, reconnect behavior,
  and stale-response isolation. Do not promise an indefinitely open iOS socket.
- Preserve the Live Activity Widget target and schemes when editing Xcode.
- For Android menus and text selection, follow `androidApp/AGENTS.md`: reuse
  the `DshMenu` family, `DshMenuHost`, and `DshMenuWebView`; do not bypass them
  with Material, platform, or custom popup menus.

## Generated Files and Local Configuration

- Do not hand-edit generated build outputs, KSP sources, gomobile artifacts,
  or ignored build directories; change their sources or build definitions.
- Treat tracked Room schema JSON as a source-defined project artifact, not an
  opaque build byproduct.
- Never run `scripts/generate_project.rb` to regenerate the current project.
  It lacks Live Activity Widget support and deliberately aborts for that
  reason. Do not remove or bypass this safeguard.
- Do not commit `local.properties`, `.env`, local signing configuration, IDE
  state, build outputs, generated AAR/XCFramework files, or machine-specific
  SDK paths.

## Security and Privileged Operations

- Never expose pairing tokens, credentials, private configuration, or sensitive
  payloads in logs, fixtures, commits, issues, or chat output.
- Keep long-lived credentials in platform secure storage. Provider API keys
  belong to the Harness host, not either mobile app.
- Use TLS and device authentication outside trusted local development.
- Preserve download size, hash, package, and signature verification. Never
  weaken certificate, authentication, transport, or integrity checks to make
  a test or connection succeed.
- Normal editing and non-destructive local validation are allowed for the task.
  Do not commit, push, publish, upload artifacts, alter deployment
  infrastructure, or change credentials/signing unless explicitly requested.

## Validation and Completion

- Add or update regression tests for shared behavior changes and bug fixes.
  Cover ordering, cancellation, stale responses, retries, and boundaries where
  relevant.
- Select tests and builds according to the affected surface. Shared changes
  require shared tests and validation of affected platform integration;
  Android-only test success does not validate an iOS bridge change.
- Use device or manual validation for behavior unit tests cannot establish,
  such as Live Activities, background execution, real Gateway interaction,
  permissions, audio, and UI interaction. Include narrow-screen, scrolling,
  dismissal, disabled-action, and selection-focus checks for menu changes.
- Inspect the final diff for accidental, unrelated, or sensitive changes.
- Report exactly what passed and what was not run, including unavailable
  toolchains, devices, or Gateway dependencies.
- Never equate compilation with behavioral verification or claim full
  verification while relevant checks are blocked. Report pre-existing failures
  separately without silently expanding scope to fix them.
