```
data Even : nat -> SProp :=
| Even_z  : Even z
| Even_ss : {n : nat} -> Even n -> Even (s (s n))
```

Pros:
- induction principle
- irrelevance (thanks SProp!)

Cons:
- quadratic proof size
- need to manually implement decision procedure
- hard to prove 1 is not even (inversion needed)
- to sum up: doesn't compute

```
even : nat -> bool :=
| z        => true
| s z      => false
| s (s n') => even n'
```

Pros:
- constant proof size
- it is its own decision procedure
- easy to prove 1 not even
- to sum up: it computes

Cons:
- no induction principle
- nonstandard shape of recursion
- hard to prove equalities like `(even n = true) = ...`, especially for beginners

So what? Let's merge both of these!

```
data EVEN : nat -> Type :=
| z        => EVEN_z : EVEN z
| s z      => False
| s (s n') => EVEN_ss : EVEN n' -> EVEN (s (s n'))
```
Pros:
- constant proof size
- easy to prove 1 not even
- it computes
- induction principle

Cons:
- need to manually implement decision procedure

Q: Can we do anything nice with this?

A: in such a banal case as parity of naturals probably not, but in more complicated ones I think so! Example: matching a regular expression against a string. This can't be easily implemented by recursion, so induction is needed. But even though we use induction, it would be nice if some cases of the definition could compute/simplify to help us a bit.

```
// Type of regexes with smart constructors built-in.
data Regex (A : Type) : Type :=
  | Empty : Regex A
  | Epsilon : Regex A
  | Char : A -> Regex A
  | Seq : Regex A -> Regex A -> Regex A :=
    | Empty       _          => Empty
    | _           Empty      => Empty
    | Epsilon     r          => r
    | r           Epsilon    => r
    | (Seq r1 r2) r3         => Seq r1 (Seq r2 r3)
    | (Or r1 r2)  r3         => Or (Seq r1 r3) (Seq r2 r3)
    | r1          (Or r2 r3) => Or (Seq r1 r2) (Seq r1 r3)
  | Or : Regex A -> Regex A -> Regex A :=
    | Empty       r          => r
    | r           Empty      => r
    | (Or r1 r2)  r3         => Or r1 (Or r2 r3)
    // TODO: Epsilon
  | Star : Regex A -> Regex A :=
    | Empty   => Epsilon
    | Epsilon => Epsilon
    | Star r  => Star r

data Matches {A : Type} (l : List A) : Regex A -> SProp :=
| Empty     => Empty
| Epsilon   => MEpsilon (p : l = [])
| Char c    => MChar (p : l = [c])
| Or r1 r2  => MOrL (m : Matches l r1) | MOrR (m : Matches l r2)
| Seq r1 r2 => MSeq (l1 l2 : list A) (p : l = l1 ++ l2)
                    (m1 : Matches l1 r1) (m2 : Matches l2 r2)
| Star r    => MStar (ms : MatchesStar l r)

/*
with MatchesStar {A : Type} : list A -> Regex A -> Prop :=
    | MS_Epsilon :
        forall r : Regex A, MatchesStar [] r
    | MS_Seq :
        forall (h : A) (t l : list A) (r : Regex A),
          Matches (h :: t) r -> MatchesStar l r -> MatchesStar ((h :: t) ++ l) r.
*/
```