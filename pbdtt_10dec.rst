Programmeren in Dependently Typed Talen
=======================================

Dependent Types
===============

* Types die afhangen van een waarde

.. class:: prettyprint lang-agda

::

    (A : Set) → (n : Nat) → Vec A n

Waarom
======

types?
------

* Voorkomen een klasse van bugs

.. class:: prettyprint lang-python

::

    a = 'tekst'
    b = True
    c = a / b

dependent types?
----------------

* Type systeem kan veel meer bugs opvangen
* Deling door 0:

.. class:: prettyprint lang-agda

::

    _div_ : (dividend : Nat)
            → (divisor : Nat)
            → (≢0 : False (divisor ≟ 0))
            → Nat

dependent types?
----------------

* Postconditie van een functie:

.. class:: prettyprint lang-agda

::

    data List (A : Set) : Set where
      []  : List A
      _∷_ : A → List A → List A

    data OList (l u : ⊥A⊤) : Set where
      nil  : l ≤̂ u → OList l u
      cons : ∀ x (xs : OList ⟦ x ⟧ u)
             → l ≤̂ ⟦ x ⟧ → OList l u

dependent types?
----------------

.. class:: prettyprint lang-agda

::

    insert : ∀ {l u} x → OList l u
             → l ≤̂ ⟦ x ⟧ → ⟦ x ⟧ ≤̂ u
             → OList l u
    insert y (nil _)         l≤y y≤u =
      cons y (nil y≤u) l≤y
    insert y (cons x xs l≤x) l≤y y≤u
      with y ≤? x
    insert y (cons x xs l≤x) l≤y y≤u
      | left  y≤x =
      cons y (cons x xs (≤-lift y≤x)) l≤y
    insert y (cons x xs l≤x) l≤y y≤u
      | right y>x =
      cons x (insert y xs
               ([ ≤-lift ,
                  (λ y≤x → absurd (y>x y≤x))
                ] (total x y)) y≤u) l≤x

dependent types?
----------------

.. class:: prettyprint lang-agda

::

    isort′ : List X → OList ⊥ ⊤
    isort′ = foldr (λ x xs → insert x xs ⊥≤̂ ≤̂⊤) (nil ⊥≤̂)

    isort : List X → List X
    isort xs = toList (isort′ xs)



Mijn thesis
===========

* Hoe verschillen talen met dependent types
* Universele patronen (views, universes)
* Patronen die taalspecifiek zijn
* Programming vs Extracting

Functors
========

Haskell
-------

.. class:: prettyprint lang-haskell

::

    class  Functor f  where
        fmap :: (a -> b) -> f a -> f b

    (<$>) :: (Functor f) => (a -> b)
             -> f a -> f b  
    f <$> x = fmap f x

    (<$)        :: a -> f b -> f a
    (<$)        =  fmap . const

Agda
----

.. class:: prettyprint lang-agda

::

    record RawFunctor (F : Set ℓ → Set ℓ) :
            Set where
      field
        _<$>_ : ∀ {A B} → (A → B)
                → F A → F B

      _<$_ : ∀ {A B} → A → F B → F A
      x <$ y = const x <$> y
