# Schmitty the Solver
```agda
{-# OPTIONS --allow-exec #-}
{-# OPTIONS --guardedness #-}

open import Data.Integer
open import Data.List
open import Data.Product
open import Relation.Binary.PropositionalEquality
open import SMT.Theories.Ints as Ints
open import SMT.Backend.Z3 Ints.reflectable
```
If you wanna solve some problems, you’re in luck! Schmitty is an Agda library which gives you bindings to SMT solvers! I know, cool right?!
```agda
verycool : ∀ (x y : ℤ) → x ≤ y → y ≤ x → x ≡ y
verycool = solveZ3
```
So, basically, what Schmitty offers you is a well-typed embedding of *some* of the SMT-LIB language in Agda. That means you can't *just* shout “solve” at your problems, you can also write SMT queries yourself!
```agda
blegh : Script [] (INT ∷ INT ∷ []) (SAT ∷ [])
blegh = `declare-const "x" INT
      ∷ `declare-const "y" INT
      ∷ `assert (`app₂ leq (# 0) (# 1))
      ∷ `assert (`app₂ leq (# 1) (# 0))
      ∷ `assert (`app₁ not (`app₂ eq (# 0) (# 1)))
      ∷ `check-sat
      ∷ []
```
Ohh, that's *almost* the script that our call to `solveZ3` above generates! What a lucky coincidence! You see, top-level constants are existentially quantified, so that script asks Z3 to see if `∃[ x ] ∃[ y ] (x ≤ y → y ≤ x → x ≢ y)` is satisfiable… and if it is, then, well, there *must* be a counter-example to our original goal!
```agda
_ : z3 blegh ≡ unsat ∷ []
_ = refl
```
Lucky us! It's *very* unsatisfiable… Wait, how did that work?! Did you just *call Z3 while type checking?!* Yes, dear reader, I did. You might’ve seen that I recently extended Agda with the `execTC` primitive, which allows you to make arbitrary system calls during type checking… well, within reason at least. Schmitty lets you take the script above, print it as an SMT-LIB term, and pass it to Z3!

Did you pick up on that `unsat` there? Schmitty doesn’t just give you back the solver’s output… she’s kind enough to actually parse the output for you! In fact, when Schmitty prints the term, she also builds you an output parser, which parses the expected solver output, including models! Let’s make sure our next query is satisfiable!
```agda
yesss : Script [] (INT ∷ INT ∷ []) (MODEL (INT ∷ INT ∷ []) ∷ [])
yesss = `declare-const "x" INT
      ∷ `declare-const "y" INT
      ∷ `assert (`app₂ leq (`app₂ sub (# 0) (# 1)) (`app₂ add (# 0) (# 1)))
      ∷ `assert (`app₁ not (`app₂ eq (# 0) (# 1)))
      ∷ `get-model
      ∷ []
```
If we call `get-model` instead of `check-sat`, Schmitty will give us back a valid model!
```agda
_ : z3 yesss ≡ ((sat , + 1 ∷ + 0 ∷ []) ∷ [])
_ = refl
```
Okay, I know that wasn’t a particularly hard problem, but I was in a rush. Send me a pull-request if you’ve got more interesting questions for Schmitty!

Wait, we can get models? Cool! We could use that to get counter-examples, if you try to prove something that *isn't* true! We, uh… We do:
```agda
woops : ∀ (x y : ℤ) → x - y ≤ x + y → x ≡ y
woops = solveZ3

-- > Found counter-example:
--     x = + 1
--     y = + 0
--   refuting (z : + 1 ≤ + 1) → + 1 ≡ + 0
--   when checking that the expression unquote solveZ3 has type
--   (x y : ℤ) → x - y ≤ x + y → x ≡ y
```

Right now, Schmitty supports three theories—[Core][SMT.Theories.Core], [Ints][SMT.Theories.Ints], and [Reals][SMT.Theories.Reals]—and two backends—[Z3][SMT.Backend.Z3], and [CVC4][SMT.Backend.CVC4]. If you’re missing your favourite theory or solver, your contribution is more than welcome!

# Installation

- [agda][agda] (>= [v2.6.2][agda-version])
- [agda-stdlib][agda-stdlib] (>= [v1.7][agda-stdlib-version])
- [agdarsec][agdarsec] ([master-03b8c4e][agdarsec-version])

Note that the path to `z3` must be added to the list of trusted executables in Agda. See  [manual.](https://agda.readthedocs.io/en/latest/language/reflection.html?highlight=trusted#system-calls)
# Roadmap

- [ ] Upstream: merge [`Text.Parser.String`][Text.Parser.String] into [agdarsec][agdarsec];
- [ ] Issue: only normalise closed subterms in error messages (moderate);
- [ ] Add error reporting to the parser (easy);
- [ ] Add backends for other SMT-LIB compliant solvers (easy);
- [ ] Add theory of real arithmetic linked to Agda rational numbers (easy);
- [ ] Add theory of floating-point numbers linked to Agda floats (easy);
- [ ] Add theory of strings linked to Agda strings (easy);
- [ ] Add theory of sequences linked to Agda lists (moderate);
- [ ] Add theory of uninterpreted functions and constants linked to Agda names (moderate);
- [ ] Add theory of regular expressions linked to [gallais/aGdaREP][aGdaREP] (moderate);
- [ ] Add theory of algebraic datatypes linked to Agda datatypes (moderate);
- [ ] Add theory of arrays linked to an axiomatisation of Haskell arrays (moderate);
- [ ] Add support for [combined theories][CombinedTheories] (moderate);
- [ ] Add support for [logic declarations][LogicDeclarations] (moderate);
- [ ] Add proof reconstruction for SAT using [`Kanso.Boolean.SatSolver`][SatSolver] (moderate);
- [ ] Add proof reconstruction for [Z3 proofs][Z3Proofs] (cf. [*Proof Reconstruction for Z3 in Isabelle/HOL*][IsabelleHol]) (hard).

[Data.Float]: https://agda.github.io/agda-stdlib/Data.Float.html
[Data.Rational]: https://agda.github.io/agda-stdlib/Data.Rational.html
[SMT.Theory]: https://wenkokke.github.io/schmitty/SMT.Theory.html
[SMT.Theories.Core]: https://wenkokke.github.io/schmitty/SMT.Theories.Core.html
[SMT.Theories.Core.Extensions]: https://wenkokke.github.io/schmitty/SMT.Theories.Core.Extensions.html
[SMT.Theories.Ints]: https://wenkokke.github.io/schmitty/SMT.Theories.Ints.html
[SMT.Theories.Reals]: https://wenkokke.github.io/schmitty/SMT.Theories.Reals.html
[SMT.Theories.Raw.Reflection]: https://wenkokke.github.io/schmitty/SMT.Theories.Raw.Reflection.html
[SMT.Script]: https://wenkokke.github.io/schmitty/SMT.Script.html
[SMT.Logics]: https://wenkokke.github.io/schmitty/SMT.Logics.html
[SMT.Backend.Z3]: https://wenkokke.github.io/schmitty/SMT.Backend.Z3.html
[SMT.Backend.CVC4]: https://wenkokke.github.io/schmitty/SMT.Backend.CVC4.html
[Text.Parser.String]: https://wenkokke.github.io/schmitty/Text.Parser.String.html
[gallais]: https://github.com/gallais
[kazkansouh]: https://github.com/kazkansouh
[satsolver]: https://github.com/wenkokke/schmitty/tree/master/extra/Kanso
[agda]: https://github.com/agda/agda
[agda-version]: https://github.com/agda/agda/releases/tag/v2.6.2
[agda-stdlib]: https://github.com/agda/agda-stdlib
[agda-stdlib-version]: https://github.com/agda/agda-stdlib/releases/tag/v1.7
[agdarsec]: https://github.com/gallais/agdarsec
[agdarsec-version]: https://github.com/gallais/agdarsec/commit/03b8c4ec57b8bc9517b5bc2fca8a540e1ec858f0
[FloatingPoint]: http://www.philipp.ruemmer.org/publications/smt-fpa.pdf
[IsabelleHol]: http://www21.in.tum.de/~boehmes/proofrec.pdf
[SatSolver]: https://github.com/wenkokke/schmitty/blob/master/extra/Kanso/Boolean/SatSolver.agda
[CombinedTheories]: http://smtlib.cs.uiowa.edu/papers/smt-lib-reference-v2.6-r2017-07-18.pdf#subsection.5.4.1
[LogicDeclarations]: http://smtlib.cs.uiowa.edu/papers/smt-lib-reference-v2.6-r2017-07-18.pdf#subsection.5.5.1
[Z3Proofs]: http://ceur-ws.org/Vol-418/paper10.pdf
[aGdaREP]: https://github.com/gallais/aGdaREP
