```agda
{-# OPTIONS --allow-unsolved-metas --exact-split --without-K --auto-inline
            --experimental-lossy-unification #-}

open import Integers.Addition renaming (_+_ to _+β€_)
open import Integers.Order
open import Integers.Integers
open import MLTT.Spartan
open import Naturals.Order
open import Notation.Order
open import UF.Base
open import UF.Equiv
open import UF.FunExt
open import UF.PropTrunc
open import UF.Quotient
open import UF.Subsingletons
open import UF.Subsingletons-FunExt

open import PLDI.DyadicRationals
open import PLDI.Prelude

module PLDI.3-TernaryBoehmRealsSearch
  (pt : propositional-truncations-exist)
  (fe : FunExt)
  (pe : PropExt)
  (sq : set-quotients-exist)
  (dy : Dyadics)
  where

open import PLDI.1-TernaryBoehmReals pt fe pe sq
open import PLDI.2-FunctionEncodings pt fe pe sq dy hiding (r)

open set-quotients-exist sq
open Dyadics dy                                   
```

# Part I - Searchable types

We first define searchable types.

```agda
decidable-predicate : {π€ π₯ : Universe} β π€ Μ β π€ β π₯ βΊ Μ
decidable-predicate {π€} {π₯} X
 = Ξ£ p κ (X β Ξ© π₯) , ((x : X) β decidable (prβ (p x)))

searchable : {π€ π₯ : Universe} (X : π€ Μ ) β π€ β π₯ βΊ Μ 
searchable {π€} {π₯} X
 = Ξ  (p , _) κ decidable-predicate {π€} {π₯} X
 , Ξ£ xβ κ X , (Ξ£ (prβ β p) β prβ (p xβ))
```

We often search only uniformly continuous predicates, which -- for general
sequence types -- are defined as follows.

```agda
_β'_ : {X : π€ Μ } β (β β X) β (β β X) β β β π€ Μ
(Ξ± β' Ξ²) n = (i : β) β i < n β Ξ± n οΌ Ξ² n
```

We could use this to define uniformly continuous predicates on π, and then
prove searchability by using the isomorphism between `π` and `β β π` to
immediately give us a searcher on such unifoormly continuous predicates using
the below properties:

```agda
decidable-predicateβ_,_ββ»ΒΉ : {X : π€ Μ } {Y : π₯ Μ } β X β Y
                         β decidable-predicate {π€} {π¦} X
                         β decidable-predicate {π₯} {π¦} Y
decidable-predicateβ e , (p , d) ββ»ΒΉ = (p β β e ββ»ΒΉ) , (d β β e ββ»ΒΉ)

decidable-predicateβ_,_β : {X : π€ Μ } {Y : π₯ Μ } β X β Y
                         β decidable-predicate {π₯} {π¦} Y
                         β decidable-predicate {π€} {π¦} X
decidable-predicateβ e , (p , d) β = (p β β e β) , (d β β e β)

decidable-predicateβ_,_β-correct
  : {X : π€ Μ } {Y : π₯ Μ } β (e : X β Y)
  β ((p , d) : decidable-predicate {π₯} {π¦} Y)
  β (y : Y) β prβ (p y)
  β prβ (prβ (decidable-predicateβ e , (p , d) β) (β e ββ»ΒΉ y))
decidable-predicateβ e , (p , d) β-correct y
 = transport (Ξ» - β prβ (p -)) (β-sym-is-rinv e _ β»ΒΉ)
              
searchableβ_,_β : {X : π€ Μ } {Y : π₯ Μ } β X β Y
                β searchable {π€} {π¦} X
                β searchable {π₯} {π¦} Y
searchableβ e , π β (p , d)
 = β e β (prβ p')
 , Ξ» (yβ , pyβ) β prβ p' ((β e ββ»ΒΉ yβ)
                , decidable-predicateβ e , (p , d) β-correct yβ pyβ)
 where p' = π (decidable-predicateβ e , (p , d) β)
```

However, the searcher given by this isomorphism (like that on signed-digits)
would search the *entire* prefix of the stream from point `pos 0` to point
`pos Ξ΄`; despite the fact that -- for ternary Boehm encodings --  the location
information at `pos Ξ΄` *includes* all of the location information previous to
that.

Therefore, we prefer to use a different isomorphism: the one induced by the
`replace` function in [`TernaryBoehmReals`](1-TernaryBoehmReals.lagda.md).

# Part II -  Searching quotiented encodings of compact intervals

First, we define the equivalence relation needed to determine uniformly
continuous decidable predicates on Ternary Boehm encodings of any compact
interval `βͺ k , i β«`.

This equivalence relation simply takes a modulus of continuity `Ξ΄ : β€` and asks
if `β¨ ΞΉ x β© Ξ΄ οΌ β¨ ΞΉ y β© Ξ΄` given `x,y : CompactInterval (k , i)`.

```agda
CompEqRel : (Ξ΄ : β€) ((k , i) : β€ Γ β€)
          β EqRel {π€β} {π€β} (CompactInterval (k , i))
CompEqRel Ξ΄ (k , i) = _β£_ , u , r , s , t
 where
   _β£_ : CompactInterval (k , i) β CompactInterval (k , i) β π€β Μ 
   (x β£ y) = β¨ ΞΉ x β© Ξ΄ οΌ β¨ ΞΉ y β© Ξ΄
   u : is-prop-valued _β£_
   u x y = β€-is-set
   r : reflexive _β£_
   r x = refl
   s : symmetric _β£_
   s x y = _β»ΒΉ
   t : transitive _β£_
   t x y z = _β_
```

Seen as we only need to look at level `Ξ΄ : β€`, we can isolate the bricks on that
level into the type `β€[ lower (k , i) Ξ΄ , upper (k , i) Ξ΄ ]`.

Indeed, the quotient type `CompactInterval (k , i) / CompEqRel Ξ΄ (k , i)` is
*equivalent* to the type `β€[ lower (k , i) Ξ΄ , upper (k , i) Ξ΄ ]`

```agda
Convβ : (Ξ΄ : β€) β ((k , i) : β€ Γ β€)
      β CompactInterval (k , i) β β€[ lower (k , i) Ξ΄ , upper (k , i) Ξ΄ ]
Convβ Ξ΄ (k , i) x = β¨ ΞΉ x β© Ξ΄ , ci-lower-upper (k , i) x Ξ΄

Convβ : (Ξ΄ : β€) β ((k , i) : β€ Γ β€)
      β β€[ lower (k , i) Ξ΄ , upper (k , i) Ξ΄ ] β CompactInterval (k , i)
Convβ Ξ΄ (k , i) (z , lβ€zβ€u) = prβ (replace (k , i) (z , Ξ΄) lβ€zβ€u)

CompReplace : (Ξ΄ : β€) ((k , i) : β€ Γ β€)
            β (x : CompactInterval (k , i))
            β x β[ CompEqRel Ξ΄ (k , i) ] (Convβ Ξ΄ (k , i) β Convβ Ξ΄ (k , i)) x
CompReplace Ξ΄ (k , i) x = prβ (replace (k , i) (z , Ξ΄) lβ€zβ€u) β»ΒΉ
 where
   Ξ³     = Convβ Ξ΄ (k , i) x
   z     = prβ Ξ³
   lβ€zβ€u = prβ Ξ³

Convβ-identifies-related-points
  : (Ξ΄ : β€) β ((k , i) : β€ Γ β€)
  β identifies-related-points {π€β} {π€β} {π€β} {CompactInterval (k , i)}
      (CompEqRel Ξ΄ (k , i)) (Convβ Ξ΄ (k , i))
Convβ-identifies-related-points Ξ΄ (k , i)
 = to-subtype-οΌ {π€β} {π€β} {β€} {Ξ» z β lower (k , i) Ξ΄ β€β€ z β€β€ upper (k , i) Ξ΄}
     (Ξ» z β β€β€Β²-is-prop {lower (k , i) Ξ΄} {upper (k , i) Ξ΄} z)

β€[_,_]-is-set : (a b : β€) β is-set (β€[ a , b ])
β€[ a , b ]-is-set = subsets-of-sets-are-sets β€ (Ξ» z β a β€β€ z β€β€ b)
                      β€-is-set (β€β€Β²-is-prop _)
                      
med-map/ : {A : π€ Μ } (Ξ΄ : β€) ((k , i) : β€ Γ β€)
         β is-set A
         β (f : CompactInterval (k , i) β A)
         β identifies-related-points (CompEqRel Ξ΄ (k , i)) f
         β CompactInterval (k , i) / CompEqRel Ξ΄ (k , i) β A
med-map/ Ξ΄ (k , i) s = mediating-map/ (CompEqRel Ξ΄ (k , i)) s

uni-tri/ : {A : π€ Μ } (Ξ΄ : β€) ((k , i) : β€ Γ β€)
         β (s : is-set A)
         β (f : CompactInterval (k , i) β A)
         β (p : identifies-related-points (CompEqRel Ξ΄ (k , i)) f)
         β med-map/ Ξ΄ (k , i) s f p β Ξ·/ (CompEqRel Ξ΄ (k , i)) βΌ f
uni-tri/ Ξ΄ (k , i) s f p = universality-triangle/ (CompEqRel Ξ΄ (k , i)) s f p

med-map : (Ξ΄ : β€) ((k , i) : β€ Γ β€)
        β CompactInterval (k , i) / CompEqRel Ξ΄ (k , i)
        β β€[ lower (k , i) Ξ΄ , upper (k , i) Ξ΄ ]
med-map Ξ΄ (k , i) = med-map/ Ξ΄ (k , i)
                      (β€[ (lower (k , i) Ξ΄) , (upper (k , i) Ξ΄) ]-is-set)
                      (Convβ Ξ΄ (k , i))
                      (to-subtype-οΌ β€β€Β²-is-prop)

uni-tri : (Ξ΄ : β€) ((k , i) : β€ Γ β€)
        β (med-map Ξ΄ (k , i) β Ξ·/ (CompEqRel Ξ΄ (k , i))) βΌ Convβ Ξ΄ (k , i)
uni-tri Ξ΄ (k , i) = uni-tri/ Ξ΄ (k , i)
                      β€[ (lower (k , i) Ξ΄) , (upper (k , i) Ξ΄) ]-is-set 
                      (Convβ Ξ΄ (k , i))
                      (to-subtype-οΌ β€β€Β²-is-prop)
           
compact-equiv : (Ξ΄ : β€) ((k , i) : β€ Γ β€)
              β CompactInterval (k , i) / CompEqRel Ξ΄ (k , i)
              β β€[ lower (k , i) Ξ΄ , upper (k , i) Ξ΄ ]
compact-equiv Ξ΄ (k , i) = f' , ((g' , fg) , (g' , gf))
 where
  f' : CompactInterval (k , i) / CompEqRel Ξ΄ (k , i)
     β β€[ lower (k , i) Ξ΄ , upper (k , i) Ξ΄ ]
  f' = med-map Ξ΄ (k , i)
  g' : β€[ lower (k , i) Ξ΄ , upper (k , i) Ξ΄ ]
     β CompactInterval (k , i) / CompEqRel Ξ΄ (k , i)
  g' = Ξ·/ (CompEqRel Ξ΄ (k , i)) β Convβ Ξ΄ (k , i)
  fg : f' β g' βΌ id
  fg (z , lβ€zβ€u)
   = uni-tri Ξ΄ (k , i) (Convβ Ξ΄ (k , i) (z , lβ€zβ€u))
   β to-subtype-οΌ β€β€Β²-is-prop (prβ (replace (k , i) (z , Ξ΄) lβ€zβ€u)) 
  gf : g' β f' βΌ id
  gf = /-induction (CompEqRel Ξ΄ (k , i)) (Ξ» _ β /-is-set (CompEqRel Ξ΄ (k , i)))
         (Ξ» y β Ξ·/-identifies-related-points (CompEqRel Ξ΄ (k , i))
           (ap (Ξ» - β β¨ ΞΉ (Convβ Ξ΄ (k , i) -) β© Ξ΄)
             (uni-tri Ξ΄ (k , i) y)
           β CompReplace Ξ΄ (k , i) y β»ΒΉ))
```

This gives us a much more efficient searcher for Ternary Boehm reals in compact
intervals, because the searcher on finite subsets of `β€` does not need to check
every element of the `π` sequence.

```agda
β€[_,_]-searchable' : (l u : β€) β (n : β) β l +pos n οΌ u
                  β searchable {π€β} {π¦} (β€[ l , u ])
β€[ l , l ]-searchable' 0 refl (p , d)
 = ((l , β€β€-refl l , β€β€-refl l))
 , Ξ» ((z , lβ€zβ€u) , pz)
   β transport (prβ β p)
       (to-subtype-οΌ β€β€Β²-is-prop ((β€β€-antisym l z lβ€zβ€u) β»ΒΉ)) pz
β€[ l , .(succβ€ (l +pos n)) ]-searchable' (succ n) refl (p , d)
 = Cases (d u*) (Ξ» pu β u* , (Ξ» _ β pu))
    (Ξ» Β¬pu β ans ,
      (Ξ» ((z , lβ€z , zβ€u) , pz)
        β Cases (β€β€-split z u zβ€u)
            (Ξ» z<u β sol ((z , lβ€z
                   , transport (z β€_) (predsuccβ€ _) (β€β€-back z u z<u))
                   , transport (prβ β p) (to-subtype-οΌ β€β€Β²-is-prop refl) pz))
            (Ξ» zοΌu β π-elim (Β¬pu (transport (prβ β p)
                                   (to-subtype-οΌ β€β€Β²-is-prop zοΌu) pz)))))
 where
   u = succβ€ (l +pos n)
   u* = u , (succ n , refl) , β€β€-refl u
   Ξ³ : β€[ l , l +pos n ] β β€[ l , u ]
   Ξ³ = β€[ l , l +pos n ]-succ
   IH = β€[ l , l +pos n ]-searchable' n refl ((p β Ξ³) , (d β Ξ³))
   ans = Ξ³ (prβ IH)
   sol = prβ IH

β€[_,_]-searchable : (l u : β€) β l β€ u β searchable {π€β} {π¦} (β€[ l , u ])
β€[ l , u ]-searchable (n , p) = β€[ l , u ]-searchable' n p

π-compact-searchable
  : ((k , i) : β€ Γ β€) (Ξ΄ : β€)
  β searchable {π€β} {π¦} (CompactInterval (k , i) / CompEqRel Ξ΄ (k , i))
π-compact-searchable (k , i) Ξ΄
 = searchableβ (β-sym (compact-equiv Ξ΄ (k , i)))
 , (β€[ (lower (k , i) Ξ΄) , (upper (k , i) Ξ΄) ]-searchable
     (lowerβ€upper (k , i) Ξ΄)) β
```

# Part III - Directly defining continuity and uniform continuity for π

We can define uniform continuity on (for example, unary) predicates on π as
follows, and show that those on compact intervals are isomorphic to a predicate
on the quotiented, searchable type considered above.

```agda
πΒΉ-uc-predicate : (π β Ξ© π¦) β π¦ Μ
πΒΉ-uc-predicate {π¦} p
 = Ξ£ Ξ΄ κ β€ , ((Ο Ξ³ : π) β β¨ Ο β© Ξ΄ οΌ β¨ Ξ³ β© Ξ΄ β p Ο holds β p Ξ³ holds)

πΒΉ-uc-predicate-ki : ((k , i) : πs) β (CompactInterval (k , i) β Ξ© π¦) β π¦ Μ
πΒΉ-uc-predicate-ki (k , i) p
   = Ξ£ Ξ΄ κ β€ , ((Ο Ξ³ : CompactInterval (k , i))
             β β¨ ΞΉ Ο β© Ξ΄ οΌ β¨ ΞΉ Ξ³ β© Ξ΄ β p Ο holds β p Ξ³ holds)

πΒΉ-uc-predicate-equiv
 : {k i : β€} β (p : CompactInterval (k , i) β Ξ© π¦)
 β ((Ξ΄ , _) : πΒΉ-uc-predicate-ki (k , i) p)
 β β! p* κ (CompactInterval (k , i) / CompEqRel Ξ΄ (k , i) β Ξ© π¦)
 , p* β Ξ·/ (CompEqRel Ξ΄ (k , i)) βΌ p
πΒΉ-uc-predicate-equiv {π¦} {k} {i} p (Ξ΄ , Ο)
 = /-universality (CompEqRel Ξ΄ (k , i))
    (Ξ©-is-set (fe π¦ π¦) (pe π¦))
    p
    (Ξ» β‘Ξ΄ β Ξ©-extensionality (fe π¦ π¦) (pe π¦)
              (prβ (Ο _ _ β‘Ξ΄))
              (prβ (Ο _ _ β‘Ξ΄)))
```

We also define continuity and uniform continuity directly on (for example,
unary) functions of type π β π.

```agda
πΒΉ-c-function : (π β π) β π€β  Μ
πΒΉ-c-function f
 = (Ο΅ : β€) (Ο : π)
 β Ξ£ Ξ΄ κ β€ , ((Ξ³ : π) β β¨ Ο β© Ξ΄ οΌ β¨ Ξ³ β© Ξ΄ β β¨ f Ο β© Ο΅ οΌ β¨ f Ξ³ β© Ο΅)

πΒΉ-uc-function-ki : ((k , i) : πs) β (CompactInterval (k , i) β π) β π€β  Μ
πΒΉ-uc-function-ki (k , i) f
 = (Ο΅ : β€)
 β Ξ£ Ξ΄ κ β€ , ((Ο Ξ³ : CompactInterval (k , i))
           β β¨ ΞΉ Ο β© Ξ΄ οΌ β¨ ΞΉ Ξ³ β© Ξ΄ β β¨ f Ο β© Ο΅ οΌ β¨ f Ξ³ β© Ο΅)
```

# Part IV - Searching function encodings on ternary Boehm encodings

We now bring in our functions as defined in
[`FunctionEncodings`](2-FunctionEncodings.lagda.md).

We eventually want to show that each function defined using the machinery in
that file yields a uniform continuity oracle that proves it is uniformly
continuous.

However, for now, we instead assume this fact, and use it to show that any
predicate `p : π β Ξ©` and function built via our machinery `f : π β π` yields
a predicate `(p β f) : π β Ξ©` that is searchable on any compact interval given
by a specific-width interval `(k , i) : πs`.

```agda
F-u-continuous
 : FunctionMachine 1 β πs β π€β  Μ 
F-u-continuous F (k , i)
 = πΒΉ-uc-function-ki (k , i) (FunctionMachine.fΜ F β [_] β ΞΉ)
  
pβ-is-uc : (F : FunctionMachine 1) {(k , i) : πs}
         β F-u-continuous F (k , i)
         β (p : π β Ξ© π¦) β πΒΉ-uc-predicate {π¦} p
         β πΒΉ-uc-predicate-ki {π¦} (k , i) (p β FunctionMachine.fΜ F β [_] β ΞΉ)
pβ-is-uc F uc p (Ξ΄ , Ο)
 = prβ (uc Ξ΄) , Ξ» Ο Ξ³ ΟΞ΄β‘Ξ³Ξ΄ β Ο (fΜ [ ΞΉ Ο ]) (fΜ [ ΞΉ Ξ³ ]) (prβ (uc Ξ΄) Ο Ξ³ ΟΞ΄β‘Ξ³Ξ΄)
 where open FunctionMachine F
```

Therefore, using the above and `πΒΉ-uc-predicate-equiv`, we have shown the method
of proving that any encoded function on π built from our machinery is searchable
on any compact interval given by a specific-width interval encoding.

We conclude here.
