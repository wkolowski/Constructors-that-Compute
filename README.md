# Type Theory Wishlist

This repo is a playground in which I'm trying to invent a better type theory/proof assistant/dependently-typed programming language. This README describes the basic syntax and type structure of the language. Each section describes a particular feature of the language by example using concrete syntax, points to relevant papers and discusses its status (like "satndard", "prototype implemented somewhere" or "unsubstantiated speculation"). The details are laid out in subdirectories (linked to by section headers), in files with the `.ttw` extension that contain commented pseudocode which shows each feature in action. At the end we point to some ideas and TODOs for later consideration. 

## The Guiding Principle of Syntax

The syntax of many ((dependently-typed) functional) languages is not optimal and sometimes not even good and, to be honest, the syntax of some of them (Coq) is horrible. We are not going to repeat their mistakes, however, by making use of a very simple principle used by programmers all over the world: Don't Repeat Yourself. To paraphrase: if we have to write something twice, the syntax is bad and has to be changed.

## Basic syntax

Comments start with `//` like in C.
```
// This is a comment.
```

Multiline comments are enclosed between ``(* *)`` like in the ML languages and can be nested. TODO: revisit this later.
```
(* A multiline (* nested *) comment. *)
```

Definitions are maximally uncluttered - no keywords, just a name, type and value.
```
name : type := value
```

Declarations, which at the top-level play the role of axioms, look similar but without the value part.
```
name : type
```

Directives that modify the language start with a `%`. They can be used to turn off termination checking, strict positivity, etc.
```
%NoTerminationCheck
wut : Empty := wut
```

## Types

| Name              | Formation        | Introduction     | Elimination      |
| ----------------- | ---------------- | ---------------- | ---------------- |
| Record types      | `(a : A, ...)`   | `(a => e, ...)`  | `p.x`            |
| Function type     | `(x : A) -> B x` | `fun x : A => e` | `f a`            |
| Inductive types   | pretty standard, see below                             |
| Coinductive types | pretty standard, see below                             |
| Empty type        | `Empty`          | impossible       | `abort`          |
| Unit type         | `Unit`           | `unit`           | not needed       |
| Universes         | `Type h p`       | `Type h p`       | impossible       |
| Strict universes  | `SType h p`      | `SType h p`      | impossible       |
| Subtype universes | `Sub A`          | implicit (?)     | implicit (?)     |
| Refinement types  | `{x : A \| P x}` | implicit (?)     | implicit (?)     |
| Path type         | `x = y`          | `path i => e`    | `p i`            |
| Nabla type        | `∇ α : A. B α`   | `ν α : A. e`     | `t @ α`          |
| Type of names     | `Name A`         | with `∇` and `ν` | pattern matching |

## [Records](Records)

Record types are the central feature of the language and they subsume modules, typeclasses, sigma types, product types, and so on (and this even extends to packaging constructs like). See below for:
- [a list of problems with records](Records/ProblemsWithRecords.md) (in Coq, but these problems occur everywhere)
- [a partial solution of these problems](Records/RecordPlayground.ttw)
- [a wild and more ambitious idea of how records should be](Records/TurboRecords.ttw)

Papers on dependent records in type theory:
- [Dependent Record Types Revisited](http://www.cs.rhul.ac.uk/home/zhaohui/DRT11.pdf)
- [Typed Operational Semantics for Dependent Record Types](http://www.cs.rhul.ac.uk/home/zhaohui/TYPES09final11-01-01.pdf)
- [Extension of Martin-Löf's Type Theory with Record Types and Subtyping](https://www.researchgate.net/publication/2683061_Extension_of_Martin-Lof's_Type_Theory_with_Record_Types_and_Subtyping)
- [Ur: Statically-Typed Metaprogramming with Type-Level Record Computation](http://adam.chlipala.net/papers/UrPLDI10/UrPLDI10.pdf)
- [Abstracting Extensible Data Types: Or, Rows by Any Other Name](https://www.pure.ed.ac.uk/ws/portalfiles/portal/87175778/Abstracting_extensible_data_types_MORRIS_DoA_tbc_VoR_CC_BY.pdf)

**Status: mostly wild speculation.**

## Functions

Function types work mostly as usual, except that we want to think that all functions (including dependent ones) take just one argument which is a big (dependent) record. This view is distinct from the usual "every function takes one argument, and then maybe returns another function").

```
add : (n m : Nat) -> Nat :=
  fun n m : Nat => ...
```

Of course we can also bind arguments together with the name, to the left of the final colon.

```
add (n m : Nat) : Nat := ...
```

Of course there is a mechanism of implicit arguments:

```
id {A : Type} (x : A) : A := x
```

Better syntax for implicit arguments (inspired by [the F* language](https://www.fstar-lang.org/)):

```
id (#A : Type) (x : A) : A := x
```

But we can also omit writing implicit arguments when they are easily inferable from something else (this piece of syntax is inspired by [Idris 2](https://idris2.readthedocs.io/en/latest/tutorial/typesfuns.html#implicit-arguments)). Example:

```
id (x : A) : A := x

comp (f : A -> B) (g : B -> C) (x : A) : C := g (f x)
```

**Status: standard.**

## Basic Inductive Types

The different classes of inductive types (enumerations, parameterized types, inductive families, etc.) have progressively more complete syntaxes, so that simple types can be written in a simple way and only the more complicated ones require more details.

Enumerations can NOT be written in a single line and must have the initial bar. Note that we don't need to (and should not) write the return type of the constructors when it's the same in every case.
```
data Bool : Type
| ff
| tt
```

Definition by patterns matching are very concise.
```
notb : Bool -> Bool
| ff => tt
| tt => ff
```

We should name the argument of each constructor, as this will be used for automatic name generation - something well known to be lacking in most proof assistants.

```
data _*_ (A B : Type) : Type
| pair (outl : A) (outr : B)
```

This doesn't affect the ordinary way of doing pattern matching that binds names.
```
swap : A * B -> B * A
| pair x y => pair y x
```

But if we want, we can rely on arguments' original names in definitions by pattern matching.
```
swap : (x : A * B) -> B * A
| pair => pair x.outr x.outl
```

We can also unpack the constructor's arguments in place to use their shorter names by postfixing the constructor's name with `{..}` (this syntax is based on Haskell's [Record Wildcards](https://kodimensional.dev/recordwildcards)).
```
swap : A * B -> B * A
| pair{..} => pair outr outl
```

Even better: we don't need to write `{..}` because records get opened/unpacked automatically.
```
swap : A * B -> B * A
| pair => pair outr outl
```

As usual, the inductive type being defined can appear as argument to any of the constructors, provided that it stands in a strictly positive position.

```
data Nat : Type
| z
| s (pred : Nat)
```

More complicated nested patterns that don't follow the type's structure are perfectly fine.

```
half : Nat -> Nat
| z       => z
| s z     => z
| s (s n) => s (half n)
```

We can use `@` to name a subpattern.
```
fib : Nat -> Nat
| z           => z
| s z         => s z
| s n1@(s n2) => fib n1 + fib n2
```

We make a distinction between **parameters**, which are bound to the left of the main colon, and **indices**, which are bound to the right of the main colon. The difference is that parameters always stay the same, so that we don't need to write them explicitly. Indices can change, so we must write them explicitly.

```
data List (A : Type) : Type
| Nil
| Cons (hd : A) (tl : List)
```

This distinction applies both to inductive and recursive definitions. It looks a bit weird at first, as that's not what people are used to, but hey, you are going to appreciate it when the definitions get more complicated!

```
map (f : A -> B) : List A -> List B :=
| [] => []
| h :: t => f h :: map t
```

The distinction between parameters and indices has some other consequences too. For example, when defining additions of naturals, the most succinct definition forces us to do it by recursion on the second argument.

```
add (n : Nat) : Nat -> Nat :=
| z    => n
| s m => s (add m)
```

For functions that are not commutative, like list append, we get a bit more headache, as we need to match two arguments even though we don't use the second one.

```
app : (l1 l2 : List A) -> List A :=
| Nil     , _ => l2
| Cons h t, _ => Cons h (app t l2)
```

In case we need to match something else besides the arguments, we can use a `with`-clause.
```
filter (p : A -> Bool) : List A -> List A
| [] => []
| h :: t with p h
  | tt => h :: filter t
  | ff => filter t
```

For inductive families, we need to explicitly write the constructor's return type (because it depends on the index), but we still don't need to write the parameters.
```
data Vec (A : Type) : Nat -> Type
| Nil  : Vec 0
| Cons : (hd : A) {n : Nat} (tl : Vec n) -> Vec (s n)
```

When doing dependent pattern matching, the shape of an earlier pattern may be determined by the shape of a later pattern, for example when we are matching on the index on an inductive family and then on an element of that family with that index. We call these _inaccessible_ patterns (following [Agda](https://agda.readthedocs.io/en/v2.5.2/language/function-definitions.html#special-patterns)).

```
head : {n : Nat} (v : Vec (s n)) -> A
| .n', Cons h n' t => h
```

**Status: standard, with Agda probably being the closest implementation to what has been described so far.**

## [Advanced Inductive Types](Induction)

Inductive families are just the tip of the iceberg, as our inductive types are supposed to be REALLY powerful. We take the usual inductive families as baseline and add:
- [Overlapping and Order-Independent Patterns](#overlapping-and-order-independent-patterns)
- [Decidable Equality Patterns](#decidable-equality-patterns)
- [Constructors that Compute](#constructors-that-compute)
- [Nominal Inductive Types](#nominal-inductive-types)
- [Induction-Induction](#induction-induction)
- [Induction-Recursion](#induction-recursion)
- [Higher Inductive Types](#higher-inductive-types)
- [Indices that Compute](#indices-that-compute)

Let's do a quick tour.

### [Overlapping and Order-Independent Patterns](Induction/OverlappingPatterns)

This is basically pattern matching on steroids. Consider the usual definitions of addition of natural numbers.

```
add : (n m : Nat) -> Nat
| z   , m => m
| s n', m => s (add n' m)
```

It's all right and good, but even though `add n z` equals `n`, it does not compute to `n`. Similarly, even though `add n (s m')` equals `s (add n m')`, it doesn't compute to `s (add n m')`. Overlapping patterns are a way to make this happen.

```
add : (n m : Nat) -> Nat
| z   , m    => m
| s n', m    => s (add n' m)
| n   , z    => n
| n   , s m' => s (add n m')
```

Here besides the two clauses from the previous definitions, we have two more clauses which amount to saying that "additionally, `add n z` computes to `z` and `add n (s m')` computes to `s (add n m')`".

For the definition to be accepted, all the clauses need to be confluent, i.e. they must return the same result when they overlap. In our example, the overlapping cases are `add z z` and `add (s n') (s m')`. But `add z z` computes to `z` both using clause 1 and 3, so it's ok. Similarly, `add (s n') (s m')` computes to `s (s (add n' m'))` using first clause 2 and then clause 4, and it computes to the same result using first clause 4 and then clause 2, so it's ok.

These new computational equalities greatly reduce the number of rewrites needed in proofs, but it also and makes dependently-typed programming much easier in some cases. For example, for vectors the terms `v ++ []` used to have the problematic type `Vec A (add n z)`, whereas now it has the less problematic type `Vec A n`. Yay!

Of course, there are some problems with this new kind of pattern matching. The foremost of them is that the catch-all pattern `_` starts being problematic. Consider the decision function for equality of naturals:

```
%Fail
dec : (n m : Nat) -> Bool
| z   , z    => tt
| s n', s m' => dec n' m'
| _   , _    => ff
```

This definition is illegal, because `dec z z` computes to `tt` according to the first clause, but it computes to `ff` according to the third clause.

To remedy this, we have to keep the old semantics of pattern matching (which we call _first-match semantics_). So from now on we have two kinds of pattern matching: the usual one with first-match semantics and the new one, in which patterns can be _overlapping_ and whose semantics don't depend on the _order_ in which the patterns appear in code. We can use the pragmas `%FirstMatch` and `%OverlappingPatterns` to specify, which kind of pattern matching we use.

```
%FirstMatch
dec : (n m : Nat) -> Bool
| z   , z    => tt
| s n', s m' => dec n' m'
| _   , _    => ff
```

Now the definition is ok, because we explicitly mark the fact that it uses the first-match semantics.

Papers:
- [Overlapping and Order-Independent Patterns: Definitional Equality for All](https://link.springer.com/content/pdf/10.1007%2F978-3-642-54833-8_6.pdf).

**Status: prototype implemented in Agda 2.6.1.**

### Decidable Equality Patterns

The idea is that for type that have decidable equality, we can have syntax sugar for non-linear patterns when pattern matching and have it translated into uses of the decision procedure for equality.

For example, we could use this feature to implement the below function which removes adjacent duplicates from a list provided that the type of elements in the list has decidable equality.

```
dedupConsecutive {A : EqType} : List A -> List A
| []          => []
| h :: h :: t => dedupConsecutive (h :: t)
| h :: t      => h :: dedupConsecutive t
```

The above desugars to
```
dedupConsecutive {A : EqType} : List A -> List A
| [] => []
| x :: y :: t with x =? y
  | tt =>      dedupConsecutive (y :: t)
  | ff => x :: dedupConsecutive (y :: t)
```

Note that the semantics of the non-linear matches are the classical first-match semantics and it looks like it'd be hard to transplant this into the setting of overlapping and order-independent patterns. 

Not papers:
- [mailing list with discussion on why non-linear patterns are not allowed in Haskell](https://www.mail-archive.com/haskell@haskell.org/msg03721.html)

**Status: no papers and nowhere implemented, but looks very easy to get right.**

### [Constructors that Compute](Induction/ConstructorsThatCompute)

The basic idea here is that during inductive type definition constructors can pattern match on their arguments and compute (almost) like ordinary recursive functions. Let's see an example.

```
data Z : Type
| z : Z
| s : Z -> Z
  | p k => k
| p : Z -> Z
  | s k => k
```

The above is a definition of the type of integers `Z` with three constructors: `z` - zero, `s`- successor, `p` - predecessor. Using ordinary inductive types this is not a good definition, because it does not represent the integers - there are terms like `s (p z)` which do not correspond to numbers. But in the above definition the constructors `s` and `p` have associated computation rules, which say that `s (p k)` computes to `k` (this is the rule for `s`) and that `p (s k)` computes to `k` (this is the rule for `p`). This means that the only legal closed normal form terms of type `Z` are `z`, finitely many `s`s applied to `z` and finitely many `p`s applied to `z`. Therefore `Z` is a good representation of the integers.

Note that as of now, the patterns allowed for constructors' computation rules are using first-match semantics, but that may change in the future.

```
abs : Z -> Z :=
| z   => z
| s k => s k
| p k => s (abs k)
```

We can define functions using pattern matching and structural recursion, just like for ordinary inductive types. We only need to handle patterns that correspond to closed terms in normal form - terms that will be "computed away" by constructors' computation rules need not (and cannot) be handled. For `z` this means that we need to handle `z`, `s k` and `p k`, but we must not handle `s (p k)` or `p (s k)`, and optionally we may separately handle `s z`, `p z` etc.

In the above example we want to compute the absolute value of the argument. For non-negative integers this is easy and we just return the argument, whereas for negative numbers we need to recursively turn predecessors into successors.

See [this file](Induction/ConstructorsThatCompute/Z.ttw) for a more thorough explanation exploration of the type of integers defined using constructors that compute.

**Status: highly experimental. It looks like if we put reasonable constraints on the kinds of computation rules associated with constructors, there isn't any abvious contradiction, nontermination or anything like that. However, there are no prototypes and no papers, except that some constructors that compute can be simulated using [Self Types](https://github.com/uwu-tech/Kind/blob/master/blog/1-beyond-inductive-datatypes.md).**

### [Nominal Inductive Types](Induction/NominalInductiveTypes/CNIC)

For every type `A` there is a type `Name A` whose elements can be thought of as "names for elements of `A`". There's also the nominal function type `∇ α : A. B` which expresses the idea of an element of `B` that may use the bound name `α` for an element of type `A`. The main use of names and the nominal function type is in conjunction with inductive types, to represent name binding in the syntax of programming languages, logics, calculi and so on.

```
data Term : Type
| Var (x : Name Term)
| App (l r : Term)
| Lam (t : ∇ α : Term. Term)
```

Representing lambda terms is easy enough. A term is either a variable which is just a name for a term wrapped in the constructor `Var`, application of one term to another, represented with `App`, or `Lam`bda abstraction, which is modeled as a term that binds a name.

```
I : Term := Lam (ν α. Var α)

K : Term := Lam (ν α. Lam (ν β. Var α))

S : Term := Lam (ν α. Lam (ν β. Lam (ν γ. App (App (Var α) (Var γ)) (App (Var β) (Var γ)))))
```

Defining `Term`s corresponding to the combinators `I`, `K` and `S` is easy, even though there's a lot of Greek letters and parentheses.

```
I : Term := λ x. x

K : Term := λ x. λ y. x

S : Term := λ x. λ y. λ z. App (App x z) (App y z)
```

Assuming we have a good notation mechanism that fuses `Lam` and `ν x.` into `λ x.` and that `Var` is declared to be a coercion from `Name Term` to `Term`, our lambda terms become very readable.

```
subst (t : Term) : (s : ∇ α : Term. Term) -> Term
| ν α. Var α => t
| ν α. Var β => Var β 
| ν α. App (l @ α) (r @ α) => App (subst l) (subst r)
| ν α. Lam (t @ α) => Lam (ν β. subst (ν α. t @ α @ β))
```

To define a function whose domain is a nominal inductive type, we can use recursion and pattern matching, but the pattern matching is a bit more powerful that usually. First, we can match on nominal function types. Second, patterns can have non-linear occurrences of elements of type `Name A`, which effectively means that we can decide equality of names.

This makes it easy to define `subst`itution of the term `t` for the free variable in `s` (`s` is of type `∇ α : Term. Term`, so we can think of it as a term with precisely one free variable). There are four cases:
- `ν α. Var α` - we have encountered the free variable, so we return `t`
- `ν α. Var β` - we have encountered some other variable, so we leave it as is
- `ν α. App (l @ α) (r @ α)` - `s` is an application of `l` to `r`, so we recurse to `subst`itute `t` in `l` and `r`
- `ν α. Lam (t @ α)` - `s` is a lambda, so we descend under the lambda while making sure not to confuse the variable bound by the lambda with the one we are looking for.

Papers:
- [The Calculus of Nominal Inductive Constructions](https://homepage.divms.uiowa.edu/~astump/papers/cinic-lfmtp09.pdf)
- [A Dependent Type Theory with Abstractable Names](https://www.sciencedirect.com/science/article/pii/S1571066115000079)

Note: so far, our nominal inductive types are based on the first of these papers, i.e. the Calculus of Nominal Inductive Constructions. There are some reasons to think that the second paper may present a better system, but so far I haven't been able to decipher it on an intuitive level.

**Status: prototype implemented for CNIC, but long ago (and with Lisp syntax, lol). Prototype implemented for FreshMLTT, but it looks like shit. No proof whether FreshMLTT has decidable typechecking.**

### [Induction-Induction](Induction/Induction-Induction)

Induction-induction allows us to simultaneously define two or more types such that the later ones can be indexed by the earlier ones.

```
data Dense (R : A -> A -> Type) : Type
| in (x : A)
| mid (x y : Dense) (H : Dense-R R x y)
| eq (x : Dense) (H : Dense-R x x) (i : I)

with Dense-R (R : A -> A -> Prop) : Dense R -> Dense R -> Prop
| in : {x y : A} (H : R x y) -> Dense-R (in x) (in y)
| midl : {x y : Dense R} (H : Dense-R x y) -> Dense-R (mid x y H) y
| midr : {x y : Dense R} (H : Dense-R x y) -> Dense-R x (mid x y H)
```

In the above example, `Dense-R R` is the dense completion of its parameter relation `R`, which means that it represents the least dense relation that contains `R`. `Dense-R` is defined at the same time as `Dense R`, which represents its carrier - the type `A` with added midpoints of all pairs `x, y` such that `R x y`.

Note that the constructors of `Dense` refer to `Dense-R`, the constructors of `Dense-R` refer to constructors of `Dense`, and the indices of `Dense-R` refer to `Dense`. This is the idea of induction-induction.

Papers:
- [Inductive-inductive definitions](http://www.cs.swan.ac.uk/~csetzer/articlesFromOthers/nordvallForsberg/phdThesisForsberg.pdf)

**Status: implemented in Agda, but absent in other mainstream languages. There are many papers which combine it with Higher Inductive Types. Probably not hard to implement. In general, looks good.** 

### [Induction-Recursion](Induction/Induction-Recursion)

Description: TODO

Papers:
-

**Status: TODO**

### [Higher Inductive Types](Induction/HIT)

Description: TODO

Papers:
- [QUOTIENTS, INDUCTIVE TYPES, & QUOTIENT INDUCTIVE TYPES](https://arxiv.org/pdf/2101.02994.pdf)
- [Type theory in a type theory with quotient inductive types](http://www.cs.nott.ac.uk/~pszgmh/kaposi-thesis.pdf)
- [Constructing Infinitary Quotient-Inductive Types](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC7788612/)
- [LARGE AND INFINITARY QUOTIENT INDUCTIVE-INDUCTIVE TYPES](https://arxiv.org/abs/2006.11736)
- [Quotient inductive-inductive types](https://arxiv.org/abs/1612.02346)
- [Codes for Quotient Inductive Inductive Types](https://akaposi.github.io/qiit.pdf)
- [Constructing quotient inductive-inductive types](https://dl.acm.org/doi/10.1145/3290315)
- [Quotient inductive-inductive definitions](http://eprints.nottingham.ac.uk/42317/1/thesis.pdf)
- [A model of type theory with quotient inductive-inductive types](http://real.mtak.hu/112971/1/1.pdf)
- [Higher Inductive Types, Inductive Families, and Inductive-Inductive Types](http://von-raumer.de/academic/phd_vonraumer.pdf)
- [A Syntax for Higher Inductive-Inductive Types](https://drops.dagstuhl.de/opus/volltexte/2018/9190/pdf/LIPIcs-FSCD-2018-20.pdf)
- [SIGNATURES AND INDUCTION PRINCIPLES FOR HIGHER INDUCTIVE-INDUCTIVE TYPES](https://lmcs.episciences.org/6100/pdf)
- [CONSTRUCTING HIGHER INDUCTIVE TYPES AS GROUPOID QUOTIENTS](https://lmcs.episciences.org/7391/pdf)
- [The Construction of Set-Truncated Higher Inductive Types](https://www.sciencedirect.com/science/article/pii/S1571066119301306)
- [Semantics of higher inductive types](https://arxiv.org/abs/1705.07088)

**Status: prototype implementations include [cubicaltt](https://github.com/mortberg/cubicaltt), Cubical Agda, Arend and some other minor languages. No general syntax for HITs is known. Various papers which describe subclasses of HITs or HITs combined with induction-induction or something like that. Probably it's very easy to get the most basic and useful HITs, but very hard to get all of them right.**

## Sum types

As for sum types, we would like to have extensible sum types, akin to OCaml's polymorphic variants. If that's not possible, then sum types are subsumed by inductive types.

Papers:
- [Abstracting Extensible Data Types: Or, Rows by Any Other Name](https://www.pure.ed.ac.uk/ws/portalfiles/portal/87175778/Abstracting_extensible_data_types_MORRIS_DoA_tbc_VoR_CC_BY.pdf) (this one is also useful for extensible records)

**Status: probably won't happen. The papers promise much, but no good implementations/prototypes, so probably there's something wrong with them.**

## [Coinductive types](Coinduction)

Coinductives should be dual to inductives, but that will be hard to achieve as they are underresearched. The minimum is to have a nice syntax sugar for "positive" coinductive types (like the coinductive duals of natural numbers and lists). Another nice thing to have would be mixed inductive-coinductived types of the form `ν X. μ Y. T`, i.e. we can define a coinductive type that has 

A possibility for handling coinductives is for them to be just (co)recursive records, but this depend on how cool and foreign records will be.

It would be nice to have a compact syntax for coinductive types. Let's try some crazy `&`s!

```
codata Stream (A : Type) : Type :=
& hd : A
& tl : Stream
```

We could then use copattern matching to define functions:

```
interleave (l r : Stream a) : Stream a :=
& hd    => l.hd
& hd tl => r.hd
& tl tl => interleave l.tl r.tl
```

So far so good, but what about coinductive lists, conatural numbers and other single-field coinductives? We shall have a syntax sugar for that! Example:

```
codata CoList (A : Type) : Type :=
| CoNil
| CoCons (hd : A) (tl : CoList)
```

The above is neither an inductive type nor a "positive" coinductive type. It is just a syntax sugar to represent something like this:

```
data CoListX (X A : Type) : Type :=
| CoNilX
| CoConsX (hd : A) (tl : X)

codata CoList (A : Type) : Type :=
& Out : CoListX CoList A

CoNil : CoList a :=
& Out => CoNilX

CoCons (h : a) (t : CoList a) : CoList a :=
& Out => CoConsX h t
```

See [this file](Coinduction/Conat.ttw) for more details on this notation.

Papers:
- TOOD

**Status: basic coinductives are standard (even though rare), the rest is speculation.**

## [Empty](TypesThatCompute/Empty.ttw) and [Unit](TypesThatCompute/Unit.ttw)

`Empty` and `Unit` are a little special in that all their terms are computationally equal, i.e. they are strict propositions, and they also enjoy special computational properties:
- `Empty + A = A`
- `A + Empty = A`
- `Empty * A = Empty`
- `A * Empty = Empty`
- `Unit * A = A`
- `A * Unit = A`
- generalizations of the above to records
- corresponding properties at the term level
- similar properties for other type formers

Relevant papers:
- [Definitional Proof-Irrelevance without K](https://hal.inria.fr/hal-01859964v2/document)
- [Type Theory Unchained: Extending Agda with User-Defined Rewrite Rules](https://drops.dagstuhl.de/opus/volltexte/2020/13066/pdf/LIPIcs-TYPES-2019-2.pdf)

Status: the universe of strict propositions is implemented in Agda and Coq. The paper proves that the theory is consistent, compatible with univalence, and has decidable typechecking. The additional computational properties can be realized using rewrite rules, whose prototype is implemented in Agda. I'm not sure how rewrite rules interact with Agda's Prop, but I think this shouldn't be a problem.

## [Types that Compute](TypesThatCompute)

Of course we don't want to confine ourselves to just built-in computational equalities for `Empty` and `Unit` - we want to be able to define custom types with custom equalities of this kind. One way to do this is with rewrite rules. See [Type Theory Unchained: Extending Agda with User-Defined Rewrite Rules](https://drops.dagstuhl.de/opus/volltexte/2020/13066/pdf/LIPIcs-TYPES-2019-2.pdf) for more on rewrite rules.

TODO:
- Find how these types will be declared.
- Make sure that it all makes sense

## [Universes](Universes/Universes.md)

We want to have a multidimensional hierarchy of universes stratified both by the usual predicative level and by homotopy level, similar to the [Arend language](https://arend-lang.github.io/about/arend-features#universe-levels). The predicative levels are basicaly naturals, whereas the homotopy levels are natural numbers extended with infinity (for untruncated types). In fact, there will be (at least) two type hierarchies: the strict one and the non-strict one.

In the strict hierarchy, `SType (h = 0)` (abbreviated `SContr`) is the universe of contractible types (whose only member is itself), `SType (h = 1)` (abbreviated `SProp`) is the universe of strict (i.e. definitionally irrelevant) propositions (like Coq's [SProp](https://coq.inria.fr/refman/addendum/sprop.html) or Agda's [Prop](https://agda.readthedocs.io/en/v2.6.0/language/prop.html)), `SType (h = 2)` (abbreviated `SSet`) is the universe of strict sets (types for which the type of paths is a strict proposition) and so on, up to `SType (h = ω)`, the universe of strict untruncated types.

The non-strict hierarchy (written simply `Type` without the "S" at the beginning) is similar. Knowing that a type has a homotopy level `h` should bring benefits which are similar but weaker than these for the strict universes.

Some reading on universes:
- [Definitional Proof-Irrelevance without K](https://hal.inria.fr/hal-01859964v2/document)
- [Generalized Universe Hierarchies and First-Class Universe Levels](https://arxiv.org/pdf/2103.00223.pdf)
- [Notes on Universes in Type Theory](http://www.cs.rhul.ac.uk/home/zhaohui/universes.pdf)

TODO:
- Work out the details of non-strict universes.
- Write some code dealing with universes.

## Subtyping and Subtype Universes

It would be nice to have subtyping, but it's hard to say how easy (or hard) it is to achieve in a dependently-typed setting.

The subtyping judgement shall be proof-relevant, i.e. it should explicitly specify the coercion used to pass from the subtype to the supertype. These coercions should be unique, i.e. there can't be two coercions from `A` to `B`. It should also be possible to declare custom coercions.

TODO: cite Luo.

Subtyping for universes is simple. We have `c : U_i <= U_j` whenever `i <= j` and the coercion `c` is a trivial lift.

Subtyping for records is a bit complicated. See [this file](Records/TurboRecords.ttw).

Subtyping for functions is contravariant, i.e. `A <= A'` and `B <= B'` implies `A' -> B <= A -> B'`.

We could make `abort : Empty -> A` and `delete : A -> Unit` into coercions if we wanted.

For sums, we basically have `A <= A + B` and `B <= A + B`.

Not sure about subtyping for inductives and coinductives.

For subtype universes, we have `A <= B` implies `Sub A <= Sub B`.

For refinement types, we have `{x : A \| P x} <= A`.

For nablas, we should have `A <= A'` and `B <= B'` implies `∇ α : A. B α <= ∇ α : A'. B' α`.

For paths types, I think `I` is invariant, so given `c : A <= B` and `p : x ={A} y` we have `path i => c (p i) : c x ={B} c y`

## [Path types and the rest of Cubical Type Theory](Paths)

We take Cubical Type Theory and the homotopical style of doing mathematics for granted. The revolution has already occurred!

But we also want to benefit from [Types that Compute](#types-that-compute) when it comes to paths, i.e. we want path characterizations like "paths between pairs are pairs of paths" to hold by computation, without needing to prove anything. See [Type Theory Unchained: Extending Agda with User-Defined Rewrite Rules](https://drops.dagstuhl.de/opus/volltexte/2020/13066/pdf/LIPIcs-TYPES-2019-2.pdf) (section 2.6) for how to accomplish something like this for Agda's usual (i.e. inductive) equality. If I read the paper correctly, it's also possible for Path types. See [here](TypesThatCompute/Paths.ttw) for some details.

TODO:
- Refresh my knowledge of and then master the machinery behind Cubical Type Theory (systems, Glue, etc.)


## Refinement types

The idea is to have, for every type `A`, the type `{x : A | P}` where `P` is some decidable strict proposition that the typechecker (or some external SMT solvers, but that's meh) can reason about. The pioneer in this space is [the F* language](https://www.fstar-lang.org/).

F* also has some additional nice features related to refinement types that make life a lot easier:
- Discriminators that check which constructor was used to make the given term, e.g. `Nil? : list 'a -> bool`, `Cons? : list 'a -> bool`
- Projections which project constructor arguments out of a term (given that the term was really made using that constructor): `Cons?.hd : l : list 'a{Cons? l} -> 'a`, `Cons?.tl : l : list 'a{Cons? l} -> list 'a`
- Note that the above are written in F* syntax and require refinement types to get anywhere.

## Modern tooling

[The Unison Language](https://www.unisonweb.org/) has a very futuristic tooling and some good ideas, including:
- codebases - Unison code is literraly stored as an AST in a nice database managed with a dedicated tool
- everything can be referred to by its hash and names are just metadata, so its easy to rename stuff and perform other similar magic like caching tests
- Unison has typed documentation which prevents it from going out of sync with the code

## Things to investigate

### Turning contexts into record types

How to turn typing contexts into record types so that we can have records in the language for free.

Beware! This is not the same idea as "first-class typing contexts" and certainly not the same as "first-class evaluation contexts".

### Global definitions/declarations

Global definitions are those that can appear in the typing context, as opposed to local definitions which can be represented by let-bindings and ultimately as just functions. Global definitions could be useful in investigating record types with already-set fields.

### Bidirectional typechecking

Bidirectional typechecking is a way of presenting the typing rules that is more algorithmic and thus better suited for implementation. It is also said to increase the quality of error messages. But most importantly, putting rules into the bidirectional format makes them less fishy.

### Typed holes

Holes are a way of leaving a part of a term unfilled as a kind of local "axiom". They can be later revisited with the help of the language's type inference, filled automatically or serve as names for goals in the proving mode. More ambitious works try to use holes for accomodating ill-typed, ill-formed and incomplete (yet unwritten) programs into the semantics.

### Typed documentation

Documentation is well known for its tendency to go out of sync with the code. So maybe it's time to make it strongly-typed?

See [the Unison Language](https://www.unisonweb.org/) for more on typed documentation.

### Explicit substitutions

Another way to make the presentation of your type theory less fishy, more concrete and down-to-earth and more amenable to implementation.

### Normal forms

How to infer, in general, an inductive characterization of normal forms from the reduction relation?

### The status of primitive constants

Primitive constants are used to include in type theory various types known from more mainstream languages, like ints, floats, arrays, etc.