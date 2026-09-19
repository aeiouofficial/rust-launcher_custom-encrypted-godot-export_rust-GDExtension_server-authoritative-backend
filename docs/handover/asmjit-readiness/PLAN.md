# AsmJit Readiness Plan — Rust/Godot Launcher + GDExtension Backend

Status: PREPARED_ONLY
Branch: `prep/asmjit-readiness-2026-09-19`
Relevance: LOW / CONDITIONAL

Rust + GDExtension already provide native compiled execution. AsmJit is not a general performance dependency.

## Activation trigger
Only evaluate if a real runtime compiler/VM or dynamically specialized CPU kernel layer becomes a product requirement.

## Planned evaluation if activated
1. Define the runtime-codegen requirement.
2. Benchmark Rust/static native baseline.
3. Define a stable C ABI boundary to a C++ AsmJit module.
4. Keep AsmJit types private to C++.
5. Validate x64/AArch64 behavior and deployment.
6. Require measurable gain before expansion.

## Non-goals
No change to encryption/export/security model, launcher behavior, GDExtension ABI, or server authority.

No implementation, PR, merge, or dependency change on this branch.
