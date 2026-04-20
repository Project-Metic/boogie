# CLAUDE.md — boogie

Boogie: intermediate verification language (IVL) and verifier used as a backend in the
Project-Metic formal verification pipeline. Boogie translates high-level proof obligations
(from Dafny and other frontends) into SMT queries dispatched to Z3 and other solvers.

Upstream: microsoft/boogie (open source, MIT license).

## What This Repo Is

The Boogie verifier — used by Project-Metic as the backend proof engine for Dafny-based
proof obligations in the binary analysis pipeline. Boogie is one of several backends; Lean 4
is the primary proof format for Metic PO series (PO-METIC-*).

## Key Invariants

- **Boogie is a backend, not the primary proof format.** Lean 4 (via Infera) is the canonical
  proof format for Metic certificates. Boogie is used for intermediate verification steps
  that produce GVR certificates.
- **Never weaken axioms to make a proof go through.** If Boogie cannot discharge an obligation,
  the correct response is to file the obligation as UNVERIFIED or to refine the proof strategy.
  Adding unsound axioms or weakening postconditions is a correctness violation.
- **Verify prover timeout behavior.** Boogie proofs that time out must be treated as UNVERIFIED,
  not as PROVED. Timeout handling must propagate the UNVERIFIED status up the certificate chain.

## Dev Commands

```bash
# Build (requires .NET SDK)
dotnet build Source/Boogie.sln

# Run tests
dotnet test Source/Boogie.sln

# Verify a Boogie program
dotnet run --project Source/BoogieDriver -- input.bpl
```

## Tech Stack

- C# (.NET), Z3 (SMT solver), Dafny (frontend)

## Integration with Project-Metic

- Boogie is invoked by `metic-api` Dafny/Boogie formal verification backend
- GVR certificates from Boogie proofs are submitted to Infera for co-signing
- Part of the `project-metic-mono` Dafny/Boogie backend path

## What NOT to Do

- Do not add unsound axioms or weaken postconditions to make proofs pass
- Do not treat a Boogie timeout as PROVED — report UNVERIFIED
- Do not use Boogie output directly as a Metic certificate without Infera co-signing
