# Acceptance gates
G0 Scope: inspect repository instructions, Cargo manifests, edition, MSRV, target/features, baseline and pinned rust_bp SHA.
G1 Style/API: fmt, Clippy, public contracts and errors; review RB01–04/RB06.
G2 Correctness: targeted unit/integration/doc/regression/property tests; actual exit status and logs.
G3 Safety/security: unsafe/FFI review, Miri where compatible, fuzzing for exposed parsers, concurrency tests, advisory and dependency review. Record exceptions.
G4 Platform/performance: native runtime on each supported target; feature matrix; benchmarks only for changed performance-sensitive paths or asserted budgets.
H1 Human maintainability (RB16–RB18): independent source comprehension, bounded change/test exercise and accurate handoff; see [mandatory contract](HUMAN_MAINTAINABILITY.md). H1 is required before G5. Human approval is not replaced by agent review.
G5 Release: independent reviewer, artifact digest, reproducible inputs, no unresolved material findings, explicit human release authorization.

Example commands (adapt features; do not blindly enable mutually exclusive features):
```sh
rustc -Vv
cargo -V
cargo fmt --all -- --check
cargo clippy --locked --workspace --all-targets -- -D warnings
cargo test --locked --workspace
cargo test --locked --workspace --doc
RUSTDOCFLAGS="-D warnings" cargo doc --locked --workspace --no-deps
cargo tree --locked
cargo audit
cargo +nightly miri test
```
Pin nightly and audit tooling for controlled execution; run Miri only for compatible targets. For published libraries also test MSRV and dependency resolution as appropriate. Test each supported feature profile separately, plus all-features only when compatible. Do not modify lockfiles without review. Record version/command, target, exit code, log reference, commit and result PASS/FAIL/BLOCKED/N/A with reason; N/A needs an applicability justification. Never declare release readiness from a scaffold.
