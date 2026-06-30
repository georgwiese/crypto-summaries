Source: [Theorem Proving in Lean 4](https://lean-lang.org/theorem_proving_in_lean4/)

# Basics

Commands:
- `#check e`: Shows the type of `e` (elaborates without evaluating)
- `#eval e`: Evaluates `e` and prints the results (compiles and runs)
- `#print x`: Shows the definition
- `variable`: For more compact function declarations. E.g. `variable (x : Nat)` and `fun (y : Nat) => x + y` means the same as `fun (x : Nat) (y : Nat) => x + y`. Variables are arguments to all definitions that reference them in the file. `section` can be used to limit the scope of variables.

Keywords:
- `def`: Declares a new constant symbol into the working environment
- `let`: Local definition. E.g `let a := t1; t2` is definitionally equal to the result of replacing every occurrence of `a` in `t2` by `t1`. Used e.g. for local variables in functions.
- `fun`: Creates a function from an expression, e.g. `fun (x : Nat) => x + 5`

**Declaration binders** are a shorthand notation for writing functions. For example, the following two definitions are equal:
```lean
def inc: Nat → Nat := fun x => x + 1
-- Adding the argument `x` to the LHS of the `:` is syntactic sugar for the
-- `fun x =>` on the RHS.
def inc (x : Nat) : Nat := x + 1
```

Definitions:
- "definitionally equal": Two terms that reduce to the same value
- "dependent types": Types that depend on parameters (including parameters of type `Type`).

# Propositions and proofs

## Propositions as types
- `Prop` is the type of propositions.
- A proposition `P : Prop` is a type itself, which can be *inhabited* (= there exists a term of type `P`) or *uninhabited* (= there is no term of type `P`).
- If there exists a term `t : P`, then `P` is true, and `t` is a proof of `P`.

Curry-Howard isomorphism:
- `Prop` is just syntactic sugar for `Sort 0` (= the bottom of the type hierarchy).
- A proof of `P` is just a term of type `P`.
- A function `P → Q` (with `P Q : Prop`) is isomorphic to the logical implication "if P then Q".
- Conjunctions `P ∧ Q` are isomorphic to product types `P × Q`.
- Disjunctions `P ∨ Q` are isomorphic to sum types `P ⊕ Q`.
- Negations `¬P` are definitionally equal to `P -> False`.

The `theorem` command is like the `def` command, since a function that takes proofs of propositions as arguments and returns a proof of another proposition is a theorem. The only difference is that proofs are marked as irreducible (not unfolded). This is fine because `Prop` enjoys "proof irrelevance": any two proofs of a proposition are definitionally equal, so the specific proof does not matter, only the fact that it is provable.

Examples:
```lean
variable {p : Prop}
variable {q : Prop}

theorem t1 (hp : p) (hq : q) : p := hp
theorem modus_ponens (hp : p) (pq : p → q) : q := pq hp
```
(The `h` in `hp` and `hq` stands for "hypothesis".)

Similarly, `have` is similar to `let`. The difference is that `have x : p := hp` does not remember the *value* of `x` (the proof), only the type (the proposition).

`sorry` is similar to Rust's `todo!()`. It is a placeholder for a proof that has not been completed yet. `axiom` is similar to `theorem`, but it does not expect a term (or, equivalently, as if the body was `sorry`). It simply postulates the existence of a term of the given type.

```lean
-- Same statement as `Classical.em`
axiom excluded_middle (p : Prop) : p ∨ ¬p

example (a: Nat): a = 1 ∨ a ≠ 1 :=
  excluded_middle (a = 1)
```

## Propositional logic
- `And`: Similar to the product type, or, equivalently, a struct storing two propositions (`And.left` and `And.right`).
- `Or`: Similar to the sum type, or, equivalently, an enum (in Lean: inductive type) with two cases (`Or.inl` and `Or.inr`). It has two constructors (to pass either one of the propositions), but only one eliminator (to handle both cases).
- `Not`: Defined as `def Not (a : Prop) : Prop := a → False`. It is a function type, so it has one constructor (the lambda) and one eliminator (application).
- `False`: An uninhabited type (no constructors), so it has no intro rules, but one elimination rule (`False.elim : False → C` for any `C`).
- `True`: An inhabited type (one constructor, no fields), so it has one intro rule (`True.intro`) but no elimination rules (it carries no information).
- `Iff`: A structure with two fields, `Iff.mp` and `Iff.mpr`, which are the two directions of the equivalence.

Introduction and elimination rules:
- An *introduction rule* says how to *build* a proof of a connective.
- An *elimination rule* says how to *use* a proof you already have.
- For an inductive type: number of constructors = number of introduction rules, and the eliminator (recursor) must cover every constructor.

| Connective | intro | elim |
|---|---|---|
| `∧` | `And.intro` | `.left`/`.right` (or `And.rec`) |
| `∨` | `Or.inl`/`Or.inr` | `Or.elim` |
| `→` | `fun` (lambda) | application |
| `∃` | `Exists.intro` | `Exists.elim` |
| `True` | `True.intro` | — (carries no info) |
| `False` | — (none) | `False.elim` |

- `∧`: one intro (one constructor), but project either side.
- `∨`: two intros, but its eliminator forces you to handle both cases.
- `True`: one intro, no fields → no useful elimination.
- `False`: zero constructors → no intro; `False.elim : False → C` gives anything.

**Shorthands**:
- If there is a single constructor and the type can be inferred, `⟨ ... ⟩` can be used as syntactic sugar for the intro rule.
- For an expression `e` of type `Foo`, `e.bar` is a shorthand for `Foo.bar e`.

**Proof examples**:
```lean
theorem and_commutative (p q: Prop) (pq: p ∧ q) : q ∧ p :=
  ⟨ pq.right, pq.left ⟩

theorem or_commutative (p q: Prop) (h: p ∨ q) : q ∨ p :=
  h.elim
    (fun (hp: p) => Or.inr hp)
    (fun (hq: q) => Or.inl hq)

theorem or_swap (p q: Prop): p ∨ q ↔ q ∨ p :=
  Iff.intro
    (fun h => or_commutative p q h)
    (fun h => or_commutative q p h)

example (p: Prop) (h: p ∨ False): p :=
  h.elim
    (fun hp => hp)
    (fun hfalse => hfalse.elim )
```

## Classical logic

In classical logic, the **law of excluded middle** states that for any proposition `p`, either `p` is true or `¬p` is true. It can be instantiated via `Classical.em`.

From this, we can derive the **double negation elimination**:
```lean
theorem double_negation (p: Prop) (h: ¬¬p) : p :=
  -- `p ∨ ¬p` follows from the law of excluded middle.
  -- In each of the two cases, we can derive `p`.
  (Classical.em p).elim
    -- If `p` is true, then we are done.
    (fun hp => hp)
    -- If `¬p` is true, we have both `hnp: ¬p` and `h: ¬¬p`, which is a contradiction
    -- from which we can derive `p` (or anything else, for that matter).
    -- `¬q` is defined as `q → False`, so `h` is a function that takes a proof of `¬p`
    -- and produces a proof of `False`. Applying the elimination rule for `False` allows
    -- us to derive any proposition, including `p`.
    -- An equivalent way to write this would be `absurd hnp h`.
    (fun hnp => (h hnp).elim )

-- The law of excluded middle also follows from double negation:
example (p: Prop): p ∨ ¬p :=
  double_negation (p ∨ ¬p) fun (h: ¬(p ∨ ¬p)) =>
    have hnp : ¬p := fun hp => h (Or.inl hp)
    h (Or.inr hnp)
```

The classical axioms give us access to additional proof patterns:
- Proof by cases: `Classical.byCases` allows us to derive `q` if we can show `p → q` and `¬p → q`.
- Proof by contradiction: `Classical.byContradiction` allows us to derive `p` if we can show `¬p → False`.

## The Universal Quantifier

If `p` is an expression, `∀ x: α, p` is syntactic sugar for `(x: α) → p`. Typically, the expression `p` will depend on `x`.

Example:
```lean
-- The following two statements are definitionally equal:
-- example (α: Type) (p q: α → Prop) : ((x: α) → p x ∧ q x) → ((y: α) → p y) :=
example (α: Type) (p q: α → Prop) : (∀ x : α, p x ∧ q x) → ∀ y : α, p y :=
    fun h: ∀ x : α, p x ∧ q x =>
    fun y: α => (h y).left
```

Often, the bound variables of a quantifier are made implicit.

```lean
variable (α: Type) (r: α → α → Prop)

variable (refl_r: ∀ {x}, r x x)
variable (symm_r: ∀ {x y}, r x y → r y x)
variable (trans_r: ∀ {x y z}, r x y → r y z → r x z)

example (a b c d: α) (hab: r a b) (hcb: r c b) (hcd: r c d): r a d :=
  trans_r (trans_r hab (symm_r hcb)) hcd
```

## Equality

The *equality* relation `Eq` is an equivalence relation (reflexive, symmetric, and transitive) with the important property that *every assertion respects the equality*, in the sense that *we can substitute equal expressions without changing the truth value*.

It has one constructor, `Eq.refl`, which states that any term is equal to itself.

```lean
example: 2 + 3 = 5 := Eq.refl 5
-- The argument to `Eq.refl` can be inferred:
example: 2 + 3 = 5 := Eq.refl _
-- The `rfl` keyword is a shorthand for `Eq.refl _`:
example: 2 + 3 = 5 := rfl
```

Given `h1 : a = b` and `h2 : p a`, we can construct a proof for `p b` using substitution: `Eq.subst h1 h2`. The `▸` macro (typed `\t`) is a shorthand for `Eq.subst`, so we can write `h1 ▸ h2` instead of `Eq.subst h1 h2`. Aside from being more concise, it works in more contexts, because it has more effective type inference heuristics.



`congrArg` is a useful tool for proving equalities. Given `h : a = b` and a function `f`, `congrArg f h` gives us `f a = f b`. It can be used to apply the same transformation to both sides of an equality.

```lean
theorem add_same (x : Nat) : x + x = 2 * x :=
  have h1 : x + x = 1 * x + 1 * x :=
    -- Nat.one_mul (n : Nat) : 1 * n = n
    -- => Using congrArg, build `x + x = 1 * x + 1 * x`
    congrArg (fun t => t + t) (Nat.one_mul x).symm
  -- (Nat.add_mul 1 1 x).symm => 1 * x + 1 * x = (1 + 1) * x
  h1.trans (Nat.add_mul 1 1 x).symm
```

Example:
```lean
example (x y : Nat) :
    (x + y) * (x + y) =
    x * x + y * x + x * y + y * y :=
  have h1 : (x + y) * (x + y) = (x + y) * x + (x + y) * y :=
    -- Nat.mul_add (n m k : Nat) : n * (m + k) = n * m + n * k
    -- => Instantiate with n = (x + y), m = x, k = y
    Nat.mul_add (x + y) x y
  have h2 : (x + y) * (x + y) = x * x + y * x + (x * y + y * y) :=
    -- Nat.add_mul (n m k : Nat) : (n + m) * k = n * k + m * k
    -- => Nat.add_mul x y x => (x + y) * x = x * x + y * x
    -- => Nat.add_mul x y y => (x + y) * y = x * y + y * y
    -- => Plugging both into h1 yields h2
    (Nat.add_mul x y x) ▸ (Nat.add_mul x y y) ▸ h1
  -- Nat.add_assoc (n m k : Nat) : n + m + k = n + (m + k)
  -- => Using this and symmetry we show that
  --    x * x + y * x + (x * y + y * y) = x * x + y * x + x * y + y * y
  -- => By transitivity, this transforms h2 to the final statement
  h2.trans (Nat.add_assoc (x * x + y * x) (x * y) (y * y)).symm
```

The `calc` keyword allows us to chain together a sequence of equalities, which is often more readable than using `trans` repeatedly.

It has the following syntax:
```
calc
  <expr>_0  'op_1'  <expr>_1  ':='  <proof>_1
  '_'       'op_2'  <expr>_2  ':='  <proof>_2
  ...
  '_'       'op_n'  <expr>_n  ':='  <proof>_n
```

The example above can be rewritten using `calc` as follows:
```lean
example (x y : Nat) :
    (x + y) * (x + y) =
    x * x + y * x + x * y + y * y :=
  calc
    (x + y) * (x + y)
      = (x + y) * x + (x + y) * y       := Nat.mul_add (x + y) x y
    _ = x * x + y * x + (x + y) * y     := (congrArg (· + (x + y) * y) (Nat.add_mul x y x))
    _ = x * x + y * x + (x * y + y * y) := (congrArg (x * x + y * x + ·) (Nat.add_mul x y y))
    _ = x * x + y * x + x * y + y * y   := (Nat.add_assoc (x * x + y * x) (x * y) (y * y)).symm
```

This becomes even more readable by using the `rw` tactic:
```lean
example (x y : Nat) :
    (x + y) * (x + y) =
    x * x + y * x + x * y + y * y :=
  calc
    (x + y) * (x + y)
      = (x + y) * x + (x + y) * y       := by rw [Nat.mul_add]
    _ = x * x + y * x + (x * y + y * y) := by rw [Nat.add_mul, Nat.add_mul]
    _ = x * x + y * x + x * y + y * y   := by rw [Nat.add_assoc (x * x + y * x) _ _]
```

## The Existential Quantifier

`∃ x : α, p x` is syntactic sugar for `Exists (fun x : α => p x)`. It has one constructor, `Exists.intro`, which takes a witness `w : α` and a proof of `p w` and gives us a proof of `∃ x : α, p x`. Its eliminator, `Exists.elim`, allows us to derive a proposition `q` from a proof of `∃ x : α, p x` if we can show that for any witness `w` and any proof of `p w`, we can derive `q`.

```lean
example (x y z : Nat) (hxy : x < y) (hyz : y < z) : ∃ w, x < w ∧ w < z :=
  -- or shorter: ⟨ y, hxy, hyz ⟩
  Exists.intro y (And.intro hxy hyz)

example (x z : Nat) (h: ∃ w, x < w ∧ w < z): x < z :=
  -- The argument to `elim` is of type `∀ (w : Nat), x < w ∧ w < z → x < z`
  -- or (equivalently): `(w : Nat) → x < w ∧ w < z → x < z`
  h.elim (fun w: Nat => fun hw: x < w ∧ w < z => Nat.lt_trans hw.left hw.right)
```

The second example can be rewritten using a `match` expression:
```lean
example (x z : Nat) (h: ∃ w, x < w ∧ w < z): x < z :=
  match h with
  | ⟨ (w: Nat), (hw: x < w ∧ w < z) ⟩ => Nat.lt_trans hw.left hw.right
```

Example:
```lean
def IsEven (a : Nat) := ∃ b, a = 2 * b

theorem even_plus_even (h1 : IsEven a) (h2 : IsEven b) : IsEven (a + b) :=
  match h1, h2 with
  | ⟨w1, (hw1: a = 2 * w1)⟩, ⟨w2, (hw2: b = 2 * w2)⟩ =>
    -- OR: have h: a + b = 2 * (w1 + w2) := by rw [hw1, hw2, Nat.mul_add]
    have h: a + b = 2 * (w1 + w2) := (
      calc a + b
          = 2 * w1 + 2 * w2 := by rw [hw1, hw2]
        _ = 2 * (w1 + w2)   := by rw [Nat.mul_add])
    ⟨ (w1 + w2), h ⟩
```

# Tactics

Tactics are a way to write proofs automatically by applying a sequence of proof steps. They are often more readable than writing out the proof term explicitly. Stating a theorem or introducing a `have` statement creates a *goal* (to construct a term of the given type). Tactics are applied to the current goal, and may produce new subgoals. The proof is complete when all goals have been solved. Wherever a term is expected, a tactic block can be used instead: `by <tactics>`, where `<tactics>` is a sequence of tactics separated by semicolons or line breaks.

## `apply` and `exact`

Example:
```lean
theorem test (p q : Prop) (hp : p) (hq : q) : p ∧ q ∧ p := by
  -- At this point, there is a single goal:
  -- ⊢ p ∧ q ∧ p
  apply And.intro
  -- `apply And.intro` consumed the goal and produced two new ones:
  -- case left
  -- ⊢ p
  -- case right
  -- ⊢ q ∧ p
  exact hp
  -- `exact hp` consumes the first goal and closes it.
  -- The `exact` tactic allows you to provide an explicit term
  -- that has the expected type
  apply And.intro
  -- Again, we split the remaining goal (q ∧ p) into 2:
  -- case right.left
  -- ⊢ q
  -- case right.right
  -- ⊢ p
  exact hq
  -- Closes right.left
  exact hp
  -- Closes right.right, which completes the proof
```

The cases are often *tagged*. In the case of the `apply` tactic, the tag names are derived from the parameters' names.
This allows you to refer to a specific subgoal by its tag name and handle them in any order:
```lean
theorem test (p q : Prop) (hp : p) (hq : q) : p ∧ q ∧ p := by
  apply And.intro
  case right =>
    apply And.intro
    case left => exact hq
    case right => exact hp
  case left => exact hp
```

Also, tactic commands can take compound expressions, so the proof above can be written more concisely as:
```lean
theorem test (p q : Prop) (hp : p) (hq : q) : p ∧ q ∧ p := by
  apply And.intro hp
  exact And.intro hq hp
```

Finally, we can also structure the proof without referring to the subgoals by name:
```lean
theorem test (p q : Prop) (hp : p) (hq : q) : p ∧ q ∧ p := by
  apply And.intro
  . exact hp
  . exact And.intro hq hp
```

Using `#print test`, we can inspect the proof term that was constructed by the tactics:
```
theorem test : ∀ (p q : Prop), p → q → p ∧ q ∧ p :=
fun p q hp hq => ⟨hp, ⟨hq, hp⟩⟩
```

## `intro`, `assumption` and `intros`

`intro x` essentially generates `fun x =>` in the proof term. You can also introduce several variables:
```lean
example (p q : Prop) : p → q → p ∧ q := by
  -- ⊢ p → q → p ∧ q
  intro hp hq
  -- `intro` introduces variables of type `p` and `q`:
  -- hp : p
  -- hq : q
  -- ⊢ p ∧ q
  exact And.intro hp hq
```

The `intro` tactic allows us to use an implicit `match`:
```lean
example (p q : α → Prop) : (∃ x, p x ∨ q x) → ∃ x, q x ∨ p x := by
  intro
  | ⟨w, Or.inl h⟩ => exact ⟨w, Or.inr h⟩
  | ⟨w, Or.inr h⟩ => exact ⟨w, Or.inl h⟩
```

The `assumption` tactic looks for a hypothesis that matches the current goal and applies it. It is equivalent to `exact h` where `h` is a hypothesis of the same type as the goal.

```lean
example (h₁ : x = y) (h₂ : y = z) (h₃ : z = w) : x = w := by
  apply Eq.trans h₁  -- Consumes `x = w`, produces new goal `y = w`
  apply Eq.trans h₂  -- Consumes `y = w`, produces new goal `z = w`
  assumption         -- Applies `h₃`
```

This is a more complicated example, involving metavariables:
```lean
example : ∀ a b c : Nat, a = b → b = c → a = c := by
  intro a b c hab hbc
  -- At this point, the goal is a = c
  apply Eq.trans
  -- `Eq.trans` has type `(h₁ : a = b) (h₂ : b = c) : a = c`, but `b`
  -- is not mentioned in the conclusion!
  -- So, Lean creates a new metavariable `?b` and 3 subgoals:
  -- case h₁: ⊢ a = ?b
  -- case h₂: ⊢ ?b = c
  -- case b: ⊢ Nat     => What is `?b`?
  assumption
  -- After this `assumption`, Lean scans the hypotheses to match ⊢ a = ?b
  -- and finds hab: a = b. To match, it unifies ?b := b.
  -- Because of this, the `b` goal is closed as a side effect.
  -- Now, `assumption` can find hbc to close the final goal
  assumption
```

With `assumption`, the names of the introduced variables are not actually referenced. The `intros` tactic (without any arguments) introduces all the variables and hypotheses in one go, without naming them.

## `rfl`

Syntactic sugar for `exact rfl`.

## `repeat`

The `repeat` combinator can be used to apply a tactic several times:

```lean
example : ∀ a b c : Nat, a = b → b = c → a = c := by
  intros
  apply Eq.trans
  repeat assumption
```

## `revert` and `generalize`

`revert` is the opposite of `intro`. It moves a hypothesis back into the goal. For example, if we have a goal `⊢ q` and a hypothesis `h : p`, then `revert h` will change the goal to `⊢ p → q` and remove `h` from the context.

Similarly, you can replace arbitrary expressions in the goal by a fresh variable using `generalize`.

## `admit`

The equivalent of `sorry` in tactic mode.

## `cases`

The `cases` is similar to `elim` in that it allows us to destructure a hypothesis.

```lean
example (p q : Prop) : p ∨ q → q ∨ p := by
  intro h
  cases h with
  | inl hp => apply Or.inr; exact hp
  | inr hq => apply Or.inl; exact hq
```

## `contradiction`

`contradiction` is a tactic that looks for a contradiction in the hypotheses and closes the goal if it finds one.

## `have`, `let`, and `show`

These tactics are very similar to their counterparts in term mode. `have` and `let` introduce a new hypothesis or definition, while `show` is a way to restate the goal.

```lean
example : 2 + 2 = 4 := by
  let n := 2
  have h : n + n = 4 := rfl
  show 2 + 2 = 4
  exact h
```

## Tactic combinators


The combinator `tactic1 <;> tactic2` applies `tactic1` to the current goal, and then applies `tactic2` to all the resulting subgoals.
```lean
example (p : Prop) : p ∨ p → p := by
  intro h
  cases h <;> assumption
```

`first | t₁ | t₂ | ... | tₙ` applies the first tactic `tᵢ` that succeeds. If all tactics fail, the whole tactic fails.

`try t` applies the tactic `t` to the current goal, and if it fails, it does nothing (it does not fail). It is equivalent to `first | t | skip`.

## `rw` (rewrite)

`rw [t]`, where `t` is a term whose type asserts an equality, rewrites the goal by replacing the left-hand side of the equality with the right-hand side. For example, `t` can be an hypothesis (`h : a = b`) or a general lemma (`add_comm: ∀ x y, x + y = y + x`). It can also be used to rewrite hypotheses (`rw [t] at h`).

`rw` example:
```lean
def divides (x y : Nat) : Prop :=
  ∃ k, k*x = y

def divides_trans (h₁ : divides x y) (h₂ : divides y z) : divides x z :=
  let ⟨k₁, (d₁: k₁ * x = y)⟩ := h₁
  let ⟨k₂, (d₂: k₂ * y = z)⟩ := h₂
  let d: (k₁ * k₂) * x = z := by rw [
    -- Goal: (k₁ * k₂) * x = z
    Nat.mul_comm k₁ k₂,
    -- Goal: (k₂ * k₁) * x = z
    Nat.mul_assoc,
    -- Goal: k₂ * (k₁ * x) = z
    d₁,
    -- Goal: k₂ * y = z
    d₂
    -- Goal: z = z => closed by `rfl` => Done!
  ]
  ⟨k₁ * k₂ , d⟩
```

## `simp` (simplify)

The `simp` tactic simplifies the goal (or hypotheses, with `simp at h`) by applying a set of rewrite rules: identities have been tagged with the `[simp]` attribute.

```lean
def f (m n : Nat) : Nat :=
  m + n + m

theorem add_self (a : Nat) : a + a = 2 * a := (Nat.two_mul a).symm

-- Option to see the trace of `simp`

set_option trace.Meta.Tactic.simp.rewrite true in
-- This simplifier proves this in these steps:
-- - Applies definition of `f` to get `a + 0 + a = 2 * a`
-- - Removes the addition of 0 to get `a + a = 2 * a`
--   (this is a rule with a `simp` attribute)
-- - Applies `add_self` to get `2 * a = 2 * a`
-- - Finishes by rfl
example (a : Nat) : f a 0 = 2 * a := by simp [f, add_self]
```

To use all the hypotheses in the context, use `simp [*]` (or `simp at *` to simplify all hypotheses).

This is an example of how to use the `simp` attribute:
```lean
def mk_symm (xs : List α) :=
  xs ++ xs.reverse

@[simp] theorem reverse_mk_symm (xs : List α) :
    (mk_symm xs).reverse = mk_symm xs := by
  simp [mk_symm]

example (xs ys : List Nat) :
    (xs ++ mk_symm ys).reverse = mk_symm ys ++ xs.reverse := by
  simp
```


## `split`

The `split` tactic is used to split if-then-else and match expressions into cases:

```lean
def f (x y z : Nat) : Nat :=
  match x, y, z with
  | 5, _, _ => y
  | _, 5, _ => y
  | _, _, 5 => y
  | _, _, _ => 1

example (x y z : Nat) : x ≠ 5 → y ≠ 5 → z ≠ 5 → z = w → f x y w = 1 := by
  intros
  simp [f]
  -- At this point, the goal is:
  -- ⊢ (match x, y, w with
  --   | 5, x, _ => y
  --   | x, 5, _ => y
  --   | x, _, 5 => y
  --   | x, _, _ => 1) =
  --   1
  split
  . contradiction
  . contradiction
  . contradiction
  . rfl

-- Or, using combinators:
example (x y z : Nat) : x ≠ 5 → y ≠ 5 → z ≠ 5 → z = w → f x y w = 1 := by
  intros; simp [f]; split <;> first | contradiction | rfl
```