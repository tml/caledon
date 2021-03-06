Caledon Language ![logo](https://raw.github.com/mmirman/caledon/hopa/media/logo.png)
====================================================================================

Caledon is a dependently typed, polymorphic, higher order logic programming language. ie, everything you need to have a conversation with your computer.

Background
----------

* This is part of my masters thesis.  Feedback would be appreciated. Considering this, it is still in the very early research stages.  Syntax is liable to change, there will be bugs, and it doesn't yet have IO (I'm still working out how to do IO cleanly, but this WILL come).

* Its named caledon after the "New Caledonian Crow" - a crow which can make tools and meta tools.  Since this language supports meta programming with holes, implicits, polymorphism, and dependent types, I thought this crow might be a good masscot. Also, file extensions are ".ncc"

* This language was inspired by twelf, haskell and agda.  

Goals
-----

* Make logic programming less repetative

* A logic programming language that is good at defining DSLs

* A language/system for conversing with the machine in a manner less one sided and instructional than regular programming.

Philosophies 
------------

* Metaprogramming should be easy and thus first class.

* User facing code should not crash - runtime code should be type checked.

* Metacode should be optionally typechecked, but well type checked.

* Metaprogramming should not require AST traversal.  

* your programming language should be turing complete - totality checking is annoying.

* Syntax should be elegant.

* Primitives should be minimal, libraries should be extensive.  Learning a culture is easy if you speak the language.  Learning a language by cultural immersion isn't as trivial.

Features
--------

* Logic programming:  Currently it uses a breadth first proof search. This is done for completeness, since the proof search is also used in type inference.  This could (and should) possibly change in the future for the running semantics of the language.  

``` 
defn num  : atom
  as zero = num
   | succ = num -> num

defn add  : num -> num -> num -> atom
  as add_zero = {N} add zero N N
   | add_succ = {N}{M}{R} add (succ N) M (succ R) <- add N M R

-- we can define subtraction from addition!
query subtract = add (succ (succ zero)) 'v (succ (succ (succ zero)))
```
* Higher order logic programming: like in twelf and lambda-prolog.  This makes HOAS much easier to do.  

``` 
defn trm : atom
  as lam = (trm -> trm) -> trm
   | app = trm -> trm -> trm

-- we can check that a term is linear!
defn linear : (trm → trm) → atom
  as linear_var = linear ( λ v . v )
   | linear_lam = {N} linear (λ v . lam (λ x . N x v)) 
                ← [x] linear (λ v . N x v)
   | linear_app1 = {V}{F} linear (λ v . app (F v) V) 
                        ← linear F
   | linear_app2 = ?∀ V . ?∀ F . linear (λ v . app F (V v)) 
                               ← linear V
```

* Polymorphism:  This isn't sound. ie, atom : atom.  The unsoundness shouldn't be too problematic, since types are used for proof search and to ensure progress and not for theorem proving.  This language doesn't support totality, worlds, or universe checking yet, since it's goal is to be a query programming language.

``` 
defn maybe   : atom → atom
  as nothing = {a} maybe a
   | just    = {a} a → maybe a
```

* Indulgent type inferring nondeterminism:  The entire type checking process is a nondeterministic search for a type check proof.  This could be massively slow, but at least it is complete.  The size of this search is bounded by the size of the types and not the whole program, so this shouldn't be too slow in practice.  (function cases should be small).  I'm working on adding search control primitives to make this more efficient.

* Holes:  types can have holes, terms can have holes.  The same proof search that is used in semantics is used in type inference, so you can use the same computational reasoning you use to program to reason about whether the type checker can infer types!  Holes get filled by a proof search on their type and the current context.  Since the entire type checking process is nondeterministic, if they get filled by a wrong term, they can always be filled again.

``` 
defn fsum_maybe  : {a}{b} (a -> b -> atom) -> maybe a -> maybe b → atom
  as fsum_nothing = {a}{b}[F : a -> b -> atom] maybe_fsum F nothing nothing
   | fsum_just    = {a}{b}[F : _ -> _ -> atom][av : a][bv : b]
                   maybe_fsum F (just av) (just bv)
                   <- F av bv
```

* Implicit arguments:  These are arguments that automagically get filled with holes when they need to be.  They form the basis for typeclasses (records to be added), although they are far more general. This is also where the language is most modern and interesting.  I'm curious to see what uses beyond typeclasses there are for these.

``` 
defn functor : (atom → atom) → atom
  as isFunctor = ∀ F . ({a}{b : _ } (a → b → atom) → F a → F b → atom) → functor F.

defn fsum : {F} functor F => {a}{b} (a → b → atom) → F a → F b → atom
  as get_fsum = [F] functor F -> [FSUM][Foo][Fa][Fb] FSUM Foo Fa Fb -> fsum Foo Fa Fb

defn functor_maybe : functor maybe -> atom.
  as is_functor_maybe = functor_maybe (isFunctor fsum_maybe).

-- this syntax is rather verbose for the moment.  I have yet to add proper records or typeclass syntax sugar.
```

* Optional unicode syntax: Monad m ⇒ ∀ t : goats . m (λ x : t . t → t).  
    * implication :  "a -> b"  or "a → b" or "a <- b"  or "a ← b" 
    * implicits:  "a => b"  or "a ⇒ b" 
    * Quantification: "[x:A] t"  or  "∀ x:A . t" or "forall x:A . t"
    * abstraction: "λ x . t" or "\x.t"
    * Quantified implicits: "{x:A} t"  or  "?∀ x:A . t" or "?forall x:A . t"


Usage
-----

* To install, cabal install

* To run, caledon file.ncc
