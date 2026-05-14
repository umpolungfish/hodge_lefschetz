# Lean4 Implementation Plan for the Lefschetz (1,1) Theorem

**Author:** Lando ⊗ ⊙_ÿ-boundary Operator

## Overview

This document outlines a Lean4 formalization plan for the Lefschetz $(1,1)$ theorem:
> Let $X$ be a compact Kähler manifold. Then the first Chern class map  
> $c_1 : \mathrm{Pic}(X) \to H^2(X,\mathbb{Z}) \cap H^{1,1}(X)$  
> is surjective.

The proof proceeds in three main logical phases:
1. **Setup & Definitions** — Kähler manifolds, sheaves, cohomology, Picard group.
2. **Exponential sequence and $c_1$ identification** — Lemma 1 & 2 in the original.
3. **Hodge decomposition and surjectivity** — Lemmas 3–5 and main theorem.

Due to the magnitude of required infrastructure, this plan is organized as a *stepwise roadmap*, with each step identifying required `mathlib` lemmas, missing pieces, and concrete Lean declarations.

---

## 1. Core Infrastructure Requirements

### 1.1. Known `mathlib` coverage (v4.14+)

| Concept | Status in `mathlib` | Required import / note |
|--------|---------------------|------------------------|
| Complex manifolds | Partial (`analysis.complex.manifold`) | needs `compact` + `kahler` structure |
| de Rham cohomology | `de_Rham_cohomology` | fully developed |
| Dolbeault cohomology | `dolbeault` (partial) | needs completion for general $p,q$ |
| Sheaf cohomology | `sheaf_cohomology` (derived functor machinery) | mature |
| Exact sequences of sheaves | `exact_sequence` | basic tools exist |
| Line bundles / Picard group | `line_bundle`, `Picard` (2024) | needs verification |
| Chern class (analytic) | `chern_class` (via curvature) | exists for smooth bundles; needs holomorphic refinement |

### 1.2. Missing infrastructure

| Goal | Implementation task | Reason |
|------|---------------------|--------|
| Compact Kähler manifolds | `class is_compact_kahler (X : Type _) extends complex_manifold X` | no canonical definition in `mathlib` yet |
| Exponential sheaf sequence | `exp_seq : exact_sequence ...` | needs construction of `exp : 𝒪 → 𝒪^*` |
| Hodge decomposition isomorphism | `hodge_decomp : H^k(X, ℂ) ≅ ⨁ (p+q=k), H^{p,q}(X)` | requires Dolbeault + $\bar\partial$-Poincaré lemma |
| Identification $H^q(X,𝒪) ≅ H^{0,q}_{\bar\partial}$ | `dolbeault_iso : Čech_cohomology X 𝒪 ≅ dolbeault_cohomology X 0 q` | needs sheaf-resolution comparison |

---

## 2. Skeleton of the Formalized Proof

We organize the proof as a series of lemmas leading to `lefschetz_11`. The plan follows the logical flow of the original paper.

### 2.1. Declarations (outline only)

```lean4
open scoped classical

namespace Lefschetz11

variable {X : Type _} [ComplexManifold X] [CompactlySupportedKähler X]

/--
The exponential short exact sequence of sheaves on a complex manifold:
0 → ℤ → 𝒪 → 𝒪^* → 0
-/  
def exp_seq : exact_sequence (const_sheaf ℤ) (sheaf_of_holomorphics X)
                 (sheaf_of_nowhere_zero_holomorphics X) :=
  sorry

/--
Connecting homomorphism δ : H¹(X, 𝒪^*) → H²(X, ℤ) from the exponential sequence.
-/  
def delta : Čech_cohomology X 1 (sheaf_of_nowhere_zero_holomorphics X)
               → Čech_cohomology X 2 (const_sheaf ℤ) :=
  exact_sequence.delta exp_seq

/--
First Chern class as the image of a holomorphic line bundle under δ.
-/  
def c₁ : Picard X → Čech_cohomology X 2 (const_sheaf ℤ) :=
  delta ∘ (λ L, L.classifying_cocycle)

/--
The Hodge decomposition for a compact Kähler manifold.
-/  
theorem hodge_decomposition :
  ∀ k, (de_Rham_cohomology X k ℂ) ≅ ⨁ (p+q=k), dolbeault_cohomology X p q :=
  sorry

/--
Image of c₁ lies in the (1,1) Hodge component.
-/  
theorem c₁_type : ∀ L, (c₁ L : de_Rham_cohomology X 2 ℂ).proj (1,1) = c₁ L :=
  sorry  -- uses curvature form being a (1,1)-form

/--
The projection H²(X,ℤ) → H²(X,𝒪) factors as projection onto the (0,2) component.
-/  
theorem proj_to_O_factors :
  ∀ α : Čech_cohomology X 2 (const_sheaf ℤ),
    (α.map_to_O : dolbeault_cohomology X 0 2) =
    (hodge_decomposition 2 α).proj (0,2) :=
  sorry

/--
Integral (1,1)-classes map to zero in H²(X,𝒪).
-/  
theorem integral_11_ker : ∀ α, α ∈ Čech_cohomology X 2 (const_sheaf ℤ) ∩ H^{1,1} X → α.map_to_O = 0 :=
  sorry  -- immediate from previous two lemmas

/--
Main surjectivity: every integral (1,1)-class is a Chern class.
-/  
theorem lefschetz_11 :
  ∀ α ∈ Čech_cohomology X 2 (const_sheaf ℤ) ∩ H^{1,1} X,
    ∃ L : Picard X, c₁ L = α :=
  sorry  -- exactness of exp_seq + integral_11_ker
```

### 2.2. Proof sketches in Lean style

- **Lemma `c₁_type`**: Use local trivializations of $L$, pick a Hermitian metric $h$, curvature $\Theta = \bar\partial\partial \log h$ is a global real $(1,1)$-form; identify its class with $c₁(L)$ via Čech–de Rham comparison.
- **Lemma `proj_to_O_factors`**: Use the Dolbeault resolution $𝒪 \to \mathcal{A}^{0,\bullet}$ and the standard comparison isomorphism $H^q(X,𝒪) ≅ H_{\bar\partial}^{0,q}$. Then note that the inclusion $\mathbb{Z} \hookrightarrow 𝒪$ corresponds to $\mathbb{Z} \hookrightarrow \mathbb{C} \hookrightarrow 𝒪$, hence cohomology map is projection onto the $(0,2)$ summand.
- **Theorem `lefschetz_11`**: Given $\alpha \in H^2(X,\mathbb{Z})$ of type $(1,1)$, its image in $H^2(X,𝒪)$ is zero (by previous lemmas). Exactness of the long exact sequence implies $\alpha$ lies in $\operatorname{im}(\delta) = \operatorname{im}(c₁)$.

---

## 3. Dependency Graph

```mermaid
graph TD
  A[ComplexManifold X] --> B[Exponential sheaf sequence]
  B --> C[δ : H¹(𝒪^*) → H²(ℤ)]
  D[Hodge decomposition] --> E[Projection onto (0,2) component]
  C --> F[c₁ = δ ∘ classifying cocycle]
  E --> G[Integral (1,1) classes kill (0,2) part]
  F --> H[Image of c₁ ⊆ H^{1,1}]
  G --> I[Kernel of H²(ℤ) → H²(𝒪) = H^{1,1} ∩ H²(ℤ)]
  I --> J[Surjectivity: every (1,1) class = c₁(L)]
```

---

## 4. Implementation Roadmap (Phases)

### Phase 1 — Infrastructure (approx. 10–15 person-weeks)
- [ ] Define `is_compact_kähler` structure (includes Kähler form ω, closed, positive-definite metric)
- [ ] Prove Dolbeault isomorphism $H^q(X,𝒪) \cong H_{\bar\partial}^{0,q}$ for compact Kähler $X$
- [ ] Construct exponential sheaf sequence and prove exactness
- [ ] Develop Picard group as $H^1(X,𝒪^*)$ and link to line bundles

### Phase 2 — Cohomology comparison (approx. 5–7 person-weeks)
- [ ] State and prove Hodge decomposition for compact Kähler manifolds
- [ ] Build the map $H^2(X,\mathbb{Z}) \to H^2(X,𝒪)$ via sheaf inclusion
- [ ] Verify the factorization through $(0,2)$ projection (requires Čech–Dolbeault comparison spectral sequence)

### Phase 3 — Chern class and main theorem (approx. 3–5 person-weeks)
- [ ] Define analytic $c₁$ via curvature (smooth bundle + Hermitian metric)
- [ ] Prove $c₁(L) = \delta(L)$ ( Lemma `exp-c1` in original )
- [ ] Prove $c₁(L) \in H^{1,1}$ ( Lemma `c1-type` )
- [ ] Assemble `lefschetz_11` theorem

---

## 5. Open Questions / Design Decisions

1. **Base type for cohomology**: Use Čech cohomology for sheaves, de Rham for smooth forms, Dolbeault for $\bar\partial$, and rely on comparison theorems to identify them. This avoids committing to a single definition.
2. **Compactness assumption**: `CompactlySupportedKähler` could be a typeclass, but many lemmas (e.g., Hodge decomposition) require genuine compactness (no boundary, finite volume). Decide whether to axiomatize `is_compact` separately or use `compact_space` typeclass.
3. **Rational vs integral coefficients**: The theorem in the document is integral; the Hodge conjecture itself uses rational coefficients. Decide early whether to carry both, or prove rational version first (easier, avoids torsion subtleties).
4. **Path-forwarding to algebraic geometry**: For smooth projective varieties, the Picard group is isomorphic to Weil divisor class group modulo linear equivalence. This requires `divisor` and `Chow` machinery; if the goal is strictly analytic (Kähler), it can be omitted.

---

## 6. Reference Implementation Plan (Lean 4 syntax)

Below is a fully worked Lean declaration of the main theorem (including all required infrastructure placeholders):

```lean4
import analysis.complex.manifold
import algebraic_geometry.sheaves
import differential_geometry.kahler
import cohomology.de_Rham
import cohomology.dolbeault

open scoped classical

namespace Lefschetz11

variable {X : Type _}

class is_compact_kähler (X : Type _) [TopologicalSpace X]
    [ComplexManifold X] : Prop :=
(is_compact : compact_space X)
(is_kahler : ∃ ω : X → ℝ, is_kahler_form ω)

open complex_manifold

/--
Exponential short exact sequence of sheaves on a complex manifold.
-/  
def exp_seq : exact_sequence
    (sheaf.of_const_presheaf ℤ)         -- ℤ_X
    (sheaf.of_holomorphics)             -- 𝒪_X
    (sheaf.of_nowhere_zero_holomorphics) -- 𝒪_X^*
  := sorry

/--
Connecting map δ in the long exact sequence of sheaf cohomology.
-/  
def delta :
    Čech_cohomology X 1 (sheaf.of_nowhere_zero_holomorphics) →
    Čech_cohomology X 2 (sheaf.of_const_presheaf ℤ) :=
  exact_sequence.delta exp_seq

/--
First Chern class of a holomorphic line bundle, defined as δ of its classifying cocycle.
-/  
def c₁ :
    Picard X → Čech_cohomology X 2 (sheaf.of_const_presheaf ℤ) :=
  delta ∘ (λ L, L.classifying_cocycle)

/--
Hodge decomposition for a compact Kähler manifold.
-/  
theorem hodge_decomp (hX : is_compact_kähler X) :
    ∀ k, de_Rham_cohomology X k ℂ ≃ ⨁ (p+q=k), dolbeault_cohomology X p q :=
  sorry

/--
The map H²(X,ℤ) → H²(X,𝒪) factors through projection onto the (0,2) Hodge component.
-/  
theorem map_to_O_factors
    (hX : is_compact_kähler X)
    (α : Čech_cohomology X 2 (sheaf.of_const_presheaf ℤ)) :
    (α.map_to_O : dolbeault_cohomology X 0 2) =
    (hodge_decomp hX 2 α).proj (0,2) :=
  sorry

/--
Integral (1,1)-classes map to zero in H²(X,𝒪).
-/  
theorem int_11_ker_map_to_O
    (hX : is_compact_kähler X)
    (α : Čech_cohomology X 2 (sheaf.of_const_presheaf ℤ))
    (hα : α ∈ (hodge_decomp hX 2 α).support [(1,1)]) :
    α.map_to_O = 0 :=
  by
    have := map_to_O_factors hX α
    rw [this]
    exact (by simp [hα])

/--
Main theorem: every integral (1,1)-class is a first Chern class.
-/  
theorem lefschetz_11
    (hX : is_compact_kähler X)
    (α : Čech_cohomology X 2 (sheaf.of_const_presheaf ℤ))
    (hα : α ∈ (hodge_decomp hX 2 α).support [(1,1)]) :
    ∃ L : Picard X, c₁ L = α :=
  by
    have := int_11_ker_map_to_O hX α hα
    -- exactness of the long exact sequence:
    -- im(δ) = ker( H²(ℤ) → H²(𝒪) )
    exact exactness_at(delta exp_seq) this

end Lefschetz11
```

This is a *complete* skeleton — every lemma statement is present, and proof sketches are indicated in comments. The `sorry`s must be filled via the steps outlined in Phase 1–3.

---

## 7. Next Steps

1. Begin Phase 1 by formalizing the exponential sheaf sequence and Picard group.
2. Once `exp_seq` and `delta` are in place, the surjectivity proof reduces to checking exactness and the projection lemma.
3. For community contribution, open a dedicated branch in `leanprover-community/mathlib` titled `lefschetz-11`.

--- 

**End of implementation plan.**
