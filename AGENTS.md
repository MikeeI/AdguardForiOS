# Repository Guidelines

## Project Overview

AdGuard for iOS is a Safari content blocker and DNS privacy app for iOS, shipped as AdGuard and AdGuard Pro variants.
It is written mainly in Swift and Objective-C on Xcode, with Ruby fastlane for build and release automation.
It combines Safari content blocking, anti-tracking, encrypted DNS (DoH, DoT, DNSCrypt), and custom filter subscriptions.

## Fork & Upstream Contribution Intent

- Official upstream: [AdguardTeam/AdguardForiOS](https://github.com/AdguardTeam/AdguardForiOS).
- This checkout is the [MikeeI/AdguardForiOS](https://github.com/MikeeI/AdguardForiOS) fork, not an independently owned product.
- The goal is to support upstream with evidence-backed issues, comments, and pull requests.
- Contribution viability (sampled 2026-10-27, last 100 closed pull requests): 1 merged pull request from a fork (#2100, 2023), 7 closed unmerged from forks; the default branch is mostly `feat: synchronize code for v…` commits.
- Prefer `Research-and-Reporting`; propose `Pull-Request-Implementation` only for a finding the user explicitly selects.
- The user also wants to find defects and understand why some files are exceptionally old.
- Prioritize defects and improvements in the app's functional behavior, especially Safari ad blocking, over interface beautification or cosmetic changes.
- Investigate the Safari ad-blocking architecture and codebase deeply, tracing rules end to end across components, dependencies, state, and failure paths rather than stopping at surface-level symptoms.
- Establish actual behavior, impact, and root cause with source, history, current upstream, and reproducible evidence where available; distinguish verified findings from hypotheses and platform-dependent gaps.
- Investigate old files using history, current usage, and upstream context; age alone does not establish a defect or justify removal.
- `ISSUES.md` provides the compact finding overview and global ID allocator.
- `issues/ISSUE-NNN.md` owns the complete durable record for one root cause.
- `FORMAT.md` owns research, drafting, implementation authorization, approval, and publication rules.
- Prefer small, well-scoped corrections with outsized maintainer or user value.
- Recommend a pull request when a bounded verified fix is ready and no active implementation owns it.
- Otherwise recommend a comment, a new issue, or continued investigation according to the evidence.
- Apply `skill-fork-contribution-tracking` for ledger, lifecycle, personal-branch, and upstream handoff work.
- Apply `skill-maintainer-communication` before external issues, pull requests, reviews, comments, or discussions.
- Search existing work first and follow current upstream templates and disclosure rules.
- Apply `skill-semantic-compression-3` when authoring or restructuring tracking content.
- Apply `skill-git-commit-format` while respecting explicit upstream contribution conventions.
- Never choose `Authorized-Work` or `Publication-Target` on the user's behalf.
- `Research-and-Reporting` permits research, issues, and comments but no source implementation.
- `Pull-Request-Implementation` authorizes only the scoped implementation recorded for that finding.
- Base upstream contribution branches on current `upstream/version/v4.5`.
- Keep fork-only context, ledgers, configuration, and personal commits out of upstream contribution diffs.
- Reproduce claimed bugs against current upstream and run the narrowest conclusive verification.
- Publish one coherent root cause per issue, comment, or pull request.

## Branch Roles

- The canonical upstream branch is `version/v4.5`: it is the upstream default and the base of the latest merged pull request (#2414, 2025-03); earlier merged pull requests used `master`.
- The `personal` branch owns fork context and contribution tracking.
- Upstream-ready source work uses separate `feat/issue-NNN`, `fix/issue-NNN`, or `chore/issue-NNN` branches off `upstream/version/v4.5`.
- Remotes: `origin` is the personal fork, `upstream` is `AdguardTeam/AdguardForiOS`.

## Finding and Contribution Ledger

- At the start of every agent session, agents MUST read root `ISSUES.md` before repository work.
- `ISSUES.md` owns the global `Next finding ID` allocator and compact cross-finding overview.
- Each `issues/ISSUE-NNN.md` owns one finding's state, evidence, drafts, and next action.
- `FORMAT.md` is authoritative for research, drafting, implementation boundaries, and publication format.
- Before adding a finding, search the index and every relevant issue record for the same symptom or root cause.
- New findings MUST use `Next finding ID`.
- Create the issue file, add its index row, and increment the allocator together.
- Finding IDs use `ISSUE-NNN`, start at `ISSUE-001`, and remain permanent.
- Never reuse, renumber, or scope IDs by subsystem, status, session, or contribution type.
- Update the issue file and `ISSUES.md` together after state, authorization, target, priority, next action, or reference changes.
- Every issue record MUST use the field and section contract in `FORMAT.md`.
- New findings start with `State: Investigating`, `Authorized-Work: Not-Selected`, and `Publication-Target: Not-Selected`.
- New findings use `External-Reference: Not published.` until an external reference exists.
- Keep findings Investigating until currentness, prior art, impact, and correction value are evidence-backed.
- Clone detectors, AST matches, text similarity, shared names, and TODOs produce candidates only.
- A duplication finding requires shared change pressure, realistic drift, and simpler consolidation.
- The user selects `Authorized-Work` for each finding.
- `Research-and-Reporting` MUST NOT implement the finding.
- `Pull-Request-Implementation` MAY implement only the recorded scope after research resolves callers and failure modes.
- Pull-request work MUST verify behavior, commit, push, and reach `PR-Ready` before publication.
- Show the exact draft and target before publishing an issue, comment, or pull request to official upstream.
- Publish to official upstream only after the user approves the exact current draft and target.
- Any draft or target change requires showing the complete current draft and target again before publication.
- Run the read-only validator bundled with `skill-fork-contribution-tracking` after every ledger mutation.
- Record the final external URL in `External-Reference` immediately after publication.
- Keep `FORMAT.md`, `ISSUES.md`, `issues/`, and fork-only `AGENTS.md` changes out of upstream contribution diffs.

### External publication approval

Only an external issue, comment, review, discussion, or pull request write is approval-gated.
Before publication, read current contribution guidance and explain applicable project policy.
The human must be able to review and own every submission statement.
Fork commits, pushes, tracking updates, and source implementation follow the active repository contract.

## Architecture & Data Flow

- `AdguardExtension/` holds the Xcode project (`Adguard.xcodeproj`, `Adguard.xcworkspace`) and app targets.
  - `AdguardApp/` is the main app (UI, ViewModel, Services, AppDelegate, localizations as `*.lproj`).
  - `AdguardPro/` is the Pro variant; `Extension/`, `ActionExtension/`, `YouTubeAdsActionExtension/`, `AdGuardTodayExtension/`, `SafariWebExtension/` are Safari/system extensions.
  - `Tunnel/` is the packet tunnel provider for DNS protection.
  - `SharedComponents/` is code shared across app and extensions; `AdguardTests/` and `TestHostApp/` hold app tests.
- `AdGuardSDK/` is the filtering core, split into `SafariAdGuardSDK` (content blockers, filter service, filter conversion, metadata, SQL scripts) and `DnsAdGuardSDK` (DNS providers, protection, filtering rules, statistics, packet tunnel); `SharedAdGuardSDK` holds common code.
- `Libs/` holds in-repo helper libraries (ACommons, ADomain, DomainParser, Networking, Utils, ABackEndClients, vendors).
- Safari ad blocking has two paths: Safari applies six declarative content-blocker JSONs; the Safari WebExtension applies advanced rules.
- `Safari/FilterService/FiltersService.swift` manages filter metadata and activation; `SharedAdGuardSDK/FilterFilesStorage.swift` stores downloaded and bundled filter texts.
- `Safari/FiltersConvertation/FiltersConverterService.swift` selects enabled filters, user rules, and one allowlist mode, subject to protection and entitlement settings.
- `Safari/FiltersConvertation/FiltersConverter.swift` routes rules by group or affinity to general, privacy, social/annoyances, security, other, and custom blockers; user rules and exceptions are added to all six.
- Paths beginning `Safari/` or `SharedAdGuardSDK/` in this section are relative to `AdGuardSDK/AdGuardSDK/`.
- `ContentBlockerConverter` from SafariConverterLib converts rules into Safari JSON and, when advanced blocking and Pro are enabled, advanced rules; the investigated version was 4.3.0, with the current pin in `AdguardSafariExtension-iOS.xcworkspace/xcshareddata/swiftpm/Package.resolved`.
- The converter enforces Safari rule limits and app-supplied iOS 17 JSON size limits; conversion results expose errors and discarded-rule counts.
- `Safari/ContentBlockers/ContentBlockersInfoStorage.swift` writes six JSONs and builds the shared advanced-rule engine; individual atomic file writes do not form a multi-file transaction.
- `Safari/SafariProtection/SafariProtection.swift` orchestrates conversion, storage, and reload; `Safari/ContentBlockers/ContentBlockerService.swift` calls `ContentBlockersManager.swift` and retries failed Safari reloads once.
- `AdguardExtension/Extension/ContentBlockerRequestHandler.swift` supplies the relevant JSON to Safari, which applies it; enabled state and saved metadata do not prove which rules Safari installed.
- The separate `AdguardExtension/SafariWebExtension/` requests URL-specific engine results through native messaging and applies CSS, extended CSS, JavaScript, and scriptlets; it requires the extension and relevant site access.
- `Safari/FiltersConvertation/FiltersConverter.swift`, `AdguardExtension/SafariWebExtension/extension/src/pages/background/engine.ts`, and `AdguardExtension/SafariWebExtension/SafariWebExtensionMessageProcessor.swift` own the advanced-rule pipeline and lookup.
- DNS filtering in `AdGuardSDK/AdGuardSDK/DNS/` and `AdguardExtension/Tunnel/` is a separate mechanism that can also block domains.
- `fastlane/` and `Builder/` contain build, signing, and release automation; `SupportingScripts/` contains helper scripts.
- The workspace is `AdguardSafariExtension-iOS.xcworkspace` at the repository root.

### Safari Ad-Blocking Investigation Leads

The following are static investigation leads, not reproduced iOS bugs or authoritative finding records.
Recheck current upstream, prior art, reachability, impact, and actual Safari behavior before treating any lead as a finding; `ISSUES.md` and `issues/` remain the finding ledger.
Prioritize the empty inverted allowlist, missing filter files, and background-update errors for reproduction.

- Empty inverted allowlist: the enabled empty list produces no inverted exception, despite the UI describing filtering only on listed sites; verify generated JSON and Safari behavior.
- Missing enabled filter: unreadable filter text without a bundled fallback is logged and omitted while conversion continues; verify whether a successful reload silently removes that filter's protection.
- Background recovery: `SafariProtection+BackgroundFetch.swift` ignores the filter-update result before conversion and reload, and existing tests expect a nil completion error in failure scenarios; verify stale filters and reported status.
- Reload and persistence: inspect partial six-file writes, background reload overlapping later writes, skipped reloads after failure, and independent app/extension queues sharing storage; verify actual installed generations and callback results.
- Advanced cache: `engine.ts` returns cached configuration before refreshing it; verify whether a policy change affects the next navigation while the background process remains alive.
- Rule limits: verify whether discarded entries can include allowlist exceptions while earlier blocking rules remain.
- Action extension: `AdguardExtension/ActionExtension/ActionExtensionUserRulesHelper.swift` discards reload callback errors and can report success; verify persisted and installed rules after reload failure.
- Advanced frames and ordering: verify non-HTTP nested-frame top-URL fallback and nondeterministic cross-blocker advanced-rule aggregation against concrete order-sensitive rules.
- Custom filters: `FiltersService.swift` stores the requested enabled state but appends an in-memory filter as enabled; normal app callers observed in the investigation add enabled filters, so verify reachability before asserting user impact.
- Historical context: `FiltersConverterService.swift` and `AffinityRulesParser.swift` remain active despite last file changes in 2021; inspect history and current usage, not age alone.
- Test evidence: `FiltersConverterTest.swift` uses obsolete APIs and is not in the Xcode test Sources phase; the included `ContentBlockersInfoStorageTest.swift` also references older APIs, but compilation requires Xcode verification.
- The initial investigation used static inspection on Linux; no iOS/Safari scenario or Xcode build was run.

## Key Directories

- `AdguardExtension/AdguardApp`: main app code.
- `AdGuardSDK/AdGuardSDK`: Safari and DNS SDK sources; `AdGuardSDK/AdGuardSDKTests`: SDK tests.
- `AdguardExtension/Tunnel`: DNS packet tunnel.
- `Libs`: shared libraries; `Libs/tests`: library tests.
- `fastlane`, `Builder`: build and release tooling.

## Development Commands

- Requires macOS with Xcode, Ruby, and bundler; Linux hosts cannot build or run the tests.
- `gem install bundler -v 2.3.9`, `bundle config set --local path '.bundle/vendor'`, `bundle install`: install fastlane.
- `bundle exec fastlane prepare`: fetch codesigning certificates (add `--env pro` for AdGuard Pro).
- `bundle exec fastlane tests`: run app unit tests (scheme from `BUILD_SCHEME`).
- `bundle exec fastlane tests_sdk`: run the `AdGuardSDKTests` scheme.
- `bundle exec fastlane build`: build for the App Store; `adhoc`, `increment`, `testflight_beta` are CI-only lanes.
- Fastlane needs an App Store Connect API key at `fastlane/AuthKey.p8`.

## Code Conventions

- Swift and Objective-C; follow the surrounding file's style.
- `classStyleTemplate.txt` is an Objective-C header template (GPL-3 license header, `#pragma mark` sections); use it for new Objective-C classes only.
- User-facing strings live in `*.lproj` localizations.
- Translations are handled by AdGuard's translation process (https://kb.adguard.com/en/general/adguard-translations); the README does not state whether translation PRs are accepted.

## Important Files

- `README.md`: build and contribution overview.
- `.github/ISSUE_TEMPLATE/`: upstream issue forms (`01-bug-report.yaml`, `02-feature-request.yaml`, `03-question.yaml`); no `CONTRIBUTING` file or pull request template exists.
- `COPYING`: license; `fastlane/Fastfile`: lane definitions.

## Runtime/Tooling Preferences

- Xcode and fastlane via bundler are the repository-owned toolchain; do not introduce other build systems.

## Testing & QA

- Tests are XCTest: `AdGuardSDK/AdGuardSDKTests` (filters, conversion, metadata, DNS SDK parts), `AdguardExtension/AdguardTests`, and `Libs/tests`.
- Running them needs macOS; on Linux, verification is limited to static source inspection.
