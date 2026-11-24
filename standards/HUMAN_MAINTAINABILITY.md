# Human maintainability and commenting — mandatory release standard

## Objective
A competent human engineer unfamiliar with the implementation must be able to understand, modify, test, build and troubleshoot the software without the original developer or AI conversation. This is a local engineering acceptance policy, informed by S08–S10 and R10–R13 in the literature matrix. No universal line-count or comment-density target is asserted.

## 1. Source navigation and naming (RB16)
- Use domain-specific names that communicate purpose and units (timeout_ms, sample_rate_hz). Reserve short names for conventional tiny scopes. Explain non-obvious abbreviations in the owning module.
- Keep control flow explicit. Prefer a named intermediate, clear match, or small function to densely nested iterator combinators or clever macros when the latter hinder understanding.
- Separate parsing, validation, domain logic, I/O and presentation where this clarifies responsibilities. Keep related logic together. Avoid needless trait layers, generic parameters, or abstractions without demonstrated benefit.
- In longer source files, use stable, searchable numbered headings for genuinely distinct logical sections (e.g. 01. Configuration, 02. Validation, 03. Execution, 04. Tests). Do not add decorative headings to tiny modules or renumber unrelated sections.
- Use rustfmt and consistent whitespace. Function length, complexity, duplication and comment ratios are review signals, not standalone pass/fail metrics.

## 2. Commenting contract (RB16)
| Comment kind | Placement | Required content |
|---|---|---|
| Module documentation | //! at module/crate entry | Responsibility, inputs/outputs, data flow, boundaries, lifecycle, link to tests where useful |
| API documentation | /// on public items | Purpose, behavior, constraints and examples; relevant failure contracts |
| Implementation rationale | // directly before a non-obvious decision | Why this choice exists, assumptions, tradeoffs, why alternative was rejected if important |
| Invariant | // at enforcement or transformation | Condition that must remain true, units, ordering, resource ownership, boundary |
| Safety | // SAFETY: immediately before unsafe block | Concrete facts establishing soundness of this operation; reference caller contract if applicable |
| Workaround | // with issue or external source | Trigger, limitation, removal condition; never undocumented magic values |
| TODO/FIXME | Near actual debt | Specific task, issue/owner, impact; no unresolved release-critical placeholder |
| Test commentary | Before non-obvious setup/assertion | Behavior or regression being protected, not a restatement of assertions |

Keep comments in direct professional English, precise and searchable. Explain why and what is not apparent from the code. Do not comment every line, narrate Rust syntax, copy a function name as a sentence, insert conversational AI commentary, or retain commented-out dead code. Update relevant docs in the same change as behavioral modifications. Preserve useful historical rationale in ADRs or commit history. Never place credentials, real personal data or private AI transcripts in comments.

## 3. Rustdoc requirements (RB17)
Document each new or materially changed public item and its containing module; establish a baseline for legacy debt. A concise summary precedes optional extended explanation. Include # Examples where useful (runnable doctest or link to shared example); # Errors for fallible API contracts; # Panics for known caller-visible panic conditions; # Safety for unsafe interfaces. Document units, valid ranges, side effects and thread/cancellation behavior when applicable. Use intra-doc links. Prefer testable examples and propagate errors with ?.
Use missing_docs for new/changed public interfaces where practical, with scoped documented exceptions and a migration plan for legacy crates.

## 4. Example: comment intent, not syntax

```rust
//! Capture input validation.
//!
//! Reject malformed sample rates before opening a device. This keeps
//! hardware initialization separate from configuration errors.

/// Validates a sample rate against the supported capture range.
///
/// # Errors
///
/// Returns an error if the rate is outside the supported range.
pub fn validate_sample_rate(sample_rate_hz: u32) -> Result<u32, &'static str> {
    const MIN_RATE_HZ: u32 = 8_000;
    const MAX_RATE_HZ: u32 = 192_000;

    // Example policy limits: reject unsupported rates before device startup
    // so the caller can report a configuration error without opening hardware.
    if !(MIN_RATE_HZ..=MAX_RATE_HZ).contains(&sample_rate_hz) {
        return Err("sample rate outside supported range");
    }

    Ok(sample_rate_hz)
}
```

The numeric limits above are illustrative, not device specifications. In production use the verified device capabilities and link the authoritative source. A bad comment would merely say "check if the rate is in range."

## 5. Repository handoff (RB18)
README: purpose, supported platform/toolchain, prerequisites, build/test/run, entry point and link to architecture.
Architecture: entry-point -> module -> domain logic -> output/error path; responsibilities and boundaries, dependency and configuration inventory, persistent data/schema/migration behavior if applicable.
ADRs: significant design choice, constraints, alternatives, rationale and consequences; reference relevant issue or commit.
Operator runbook: config example without secrets, startup/shutdown, logs, failure diagnosis, artifact/version identification, upgrade and rollback.
Tests: link important behaviors to regression fixtures and exact command/feature/target.
Generated files: authoritative source, tool/version and reproducible regeneration command; never require a human to hand-edit generated output.
Keep documentation close to code, avoid duplicated instructions, and version the docs with the implementation.

## 6. Independent human comprehension gate H1
Required for each materially changed component; documentation maintenance alone may use a scoped review. The independent reviewer (not the implementing agent) must record:
1. Locate the relevant entry point, implementation, configuration and tests from repository sources without AI conversation context.
2. Explain the normal flow, failure flow, important invariants and reason for non-obvious design decisions.
3. Identify the intended edit location and affected interfaces. Perform one bounded change or configuration exercise in a disposable worktree and run the associated test, then discard the review-only change. For a source-only change, use a bounded trace/review instead and mark executable checks not applicable only with rationale.
4. Verify that relevant comments remain accurate, rustdoc/examples build, and the documented build/run instructions match actual execution where supported.
5. Record reviewer identity, base SHA, policy SHA, command/log, files and findings. Actual human sign-off is required before production release; agent review is preparation, not human sign-off.

PASS requires all applicable checks to have evidence and no unresolved material comprehension defect. FAIL for misleading docs or an untraceable critical flow; BLOCKED for missing reviewer, runtime or hardware. Legacy debt requires scoped exceptions with owner, expiry, issue and compensating control. No blanket waivers; H1 cannot be bypassed by passing Clippy.
