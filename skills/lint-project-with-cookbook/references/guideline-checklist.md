# Agentic Cookbook Guideline Checklist

Quick-reference checklist for reviewing implementations against the agentic cookbook.
Read `cookbook/guidelines/INDEX.md` for the full guideline index.

## UI Quality & Behavior

- [ ] **Native controls** (cookbook/principles/native-controls.md) — Platform built-in controls used first
- [ ] **Open-source preference** (cookbook/principles/open-source-preference.md) — Proven libraries before custom
- [ ] **Responsiveness & progress** (cookbook/guidelines/general.md — Always show progress) — UI reacts immediately, progress shown
- [ ] **Main thread** (cookbook/guidelines/general.md — No blocking the main thread) — No blocking on main thread
- [ ] **Deep linking** (cookbook/guidelines/general.md — Deep linking) — Significant views deep-linkable
- [ ] **Scriptability** (cookbook/guidelines/general.md — Scriptable and automatable) — Automatable where applicable
- [ ] **Accessibility display options** (cookbook/guidelines/general.md — Respect accessibility display options) — Reduce motion, contrast, etc.

## Quality Assurance

- [ ] **Unit testing** (cookbook/guidelines/general.md — Comprehensive unit testing) — Comprehensive, cover logic/state/edge cases
- [ ] **Design decisions** (cookbook/guidelines/general.md — Surface all design decisions) — LLM decisions surfaced and recorded
- [ ] **Atomic commits** (cookbook/guidelines/code-quality/atomic-commits.md) — Build-verify-commit loop, one logical change each
- [ ] **Scope discipline** (cookbook/guidelines/code-quality/scope-discipline.md) — Changes limited to stated scope, no adjacent refactoring
- [ ] **Bulk verification** (cookbook/guidelines/code-quality/bulk-operation-verification.md) — Stale reference check after multi-file operations
- [ ] **Verification** (cookbook/guidelines/testing/post-generation-verification.md) — Build, test, lint, a11y, review
- [ ] **Logging** (cookbook/guidelines/general.md — Instrumented logging) — Matches recipe's Logging section exactly
- [ ] **Feature flags** (cookbook/guidelines/general.md — Feature flags) — Behind flags via interface
- [ ] **Analytics** (cookbook/guidelines/general.md — Analytics) — Events match recipe's Analytics section
- [ ] **A/B testing** (cookbook/guidelines/general.md — A/B testing) — Variant support where applicable
- [ ] **Debug mode** (cookbook/guidelines/general.md — Debug mode) — Debug panel, not in release
- [ ] **Linting** (cookbook/guidelines/general.md — Linting from day one) — Platform linter configured and passing

## Accessibility, i18n, Privacy

- [ ] **Accessibility** (cookbook/guidelines/general.md — Accessibility from day one) — Roles, labels, keyboard, Dynamic Type, contrast
- [ ] **Localization** (cookbook/guidelines/general.md — Localizability) — All strings localized
- [ ] **RTL support** (cookbook/guidelines/general.md — RTL layout support) — Leading/trailing, tested with RTL
- [ ] **Privacy** (cookbook/guidelines/general.md — Privacy and security by default) — Data minimization, no PII in logs, secure storage

## Engineering Principles

All files in `cookbook/principles/`:
- [ ] Simplicity — No braided concerns, no unnecessary abstraction
- [ ] Work → Right → Fast — Correct first, then clean, then optimized
- [ ] Composition over inheritance — Protocols/interfaces preferred
- [ ] Dependency injection — Services injected via protocol/interface
- [ ] Immutability by default — let/val/const preferred
- [ ] Fail fast — Invalid state detected immediately
- [ ] Idempotency — Actions safe to repeat
- [ ] Design for deletion — Easy to remove without cascading changes
- [ ] YAGNI — No speculative features
- [ ] Explicit over implicit — No hidden behavior
- [ ] Small, reversible decisions — Incremental, not binding
- [ ] Tight feedback loops — Fast tests, fast builds
- [ ] Separation of concerns — One concept per module
- [ ] Least astonishment — APIs and UI behave as expected
- [ ] Manage complexity through boundaries — Clean interfaces
- [ ] Meta-principle: Optimize for change — Favor adaptability
