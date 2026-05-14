# Lefschetz (1,1) Theorem — Lean4 Formalization Plan

## Summary

This directory contains a high-level implementation plan for formally verifying the Lefschetz (1,1) theorem in Lean 4.

### Main Result (to be formalized)

> **Theorem (Lefschetz (1,1)).** Let $X$ be a compact Kähler manifold. Then the first Chern class map  
> $$c_1 : \mathrm{Pic}(X) \longrightarrow H^2(X,\mathbb{Z}) \cap H^{1,1}(X)$$  
> is surjective. In particular, every integral $(1,1)$-class on $X$ is the first Chern class of a holomorphic line bundle, hence algebraic when $X$ is projective.

### Structure of the Formalization

| Section | Lean file (proposed) | Description |
|---------|----------------------|-------------|
| `Foundations.lean` | `foundations/` | Compact Kähler manifolds, sheaves, Čech/Dolbeault/de Rham cohomology |
| `Exponential.lean` | `exponential/` | Exponential sequence, δ map, Picard group, $c_1$ definition |
| `Hodge.lean` | `hodge/` | Hodge decomposition, projection to $(p,q)$ components |
| `Main.lean` | `main/` | Lemmas on vanishing in $H^{0,2}$, surjectivity, theorem statement |

### Dependencies

- **Core libraries**: `mathlib` v4.14+ (sheaf_cohomology, de_Rham_cohomology, complex_manifold, kahler)
- **Missing infrastructure** (to be added to mathlib):
  - `is_compact_kähler` typeclass
  - Dolbeault isomorphism $H^q(X, \mathcal{O}_X) \cong {H_{\bar\partial}^{0,q}}$
  - Explicit exponential sheaf sequence and its exactness proof

### Roadmap

1. **Phase 1 (infrastructure)** — Define compact Kähler manifolds, construct exponential sequence.
2. **Phase 2 (cohomology comparison)** — Prove Hodge decomposition and projection factorization.
3. **Phase 3 (main proof)** — Assemble lemmas into `lefschetz_11`.

### Citation

The proof follows the exposition in:

- P. Griffiths & J. Harris, *Principles of Algebraic Geometry*, Wiley, 1978.
- C. Voisin, *Hodge Theory and Complex Algebraic Geometry I*, Cambridge UP, 2002.

### UNLICENSE

UNLICENSE

---
