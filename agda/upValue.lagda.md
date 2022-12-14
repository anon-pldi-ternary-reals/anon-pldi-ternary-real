```agda
{-# OPTIONS --allow-unsolved-metas --exact-split --without-K --auto-inline
            --experimental-lossy-unification #-}
            
open import Integers.Order
open import Integers.Integers
open import MLTT.Spartan
open import Naturals.Order
open import Naturals.Properties
open import Notation.Order

open import PLDI.Prelude

module PLDI.upValue where

ceilog2-type : ๐คโ ฬ
ceilog2-type
 = (n : โ)
 โ ฮฃ m ๊ โ , 2 โ^ m < (succ (succ n)) ร (succ (succ n)) โค 2 โ^ (succ m)

-- (ceilog2 n refers to ceiling log2 of (n - 2))
ceilog2 : ceilog2-type
ceilog2 0        = 0 , โ , โ
ceilog2 (succ n) with ceilog2 n
... | m , lโ , lโ with โค-split (succ (succ (succ n))) (2 โ^ succ m) lโ
... | inl is-less
 = m
 , (<-trans (2 โ^ m) (succ (succ n)) (succ (succ (succ n))) lโ (<-succ n))
 , is-less
... | inr is-equal = succ m , I , II
 where
  I : 2 โ^ succ m โคโ succ (succ n)
  I = transport (2 โ^ succ m โค_) i (โค-refl (2 โ^ (succ m)))
   where
    i : 2 โ^ succ m ๏ผ succ (succ n)
    i = succ-lc is-equal โปยน
  II : succ (succ (succ n)) โคโ (2 โ^ succ (succ m))
  II = transport (_โค 2 โ^ succ (succ m)) (is-equal โปยน)
         (exponents-of-two-ordered (succ m))

-- fun x = ceil(log2(x+1))

clog2 : โ โ โ
clog2 0 = 0
clog2 (succ n) = succ (prโ (ceilog2 n))

upValue : (a b : โค) โ a โค b โ โ
upValue a b (n , aโคb) = clog2 (pred (pred n))
```
