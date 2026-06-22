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

# Appendix

TODO:
- Explain `show`, `suffices`