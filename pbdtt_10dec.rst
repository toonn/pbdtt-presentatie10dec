Programmeren in Dependently Typed Talen
=======================================
Toon Nolten

.. image:: image/cc_by.png


Overzicht
=========
* Dependent types

  * Wat
  * Waarom

* Mijn thesis

  * Vergelijking van dependently typed talen
  * Technieken die vaak gebruikt worden
  * Programming vs Extracting


Dependent Types
===============

* Types die afhangen van een waarde

.. code-block:: agda

    data Vec (A : Set) : ℕ → Set where
      []  : Vec A zero
      _∷_ : {n : ℕ} → A → Vec A n → Vec A (suc n)

    head : {A : Set} → {n : ℕ} → Vec A (suc n) → A
    head (x ∷ xs) = x


Waarom
======
types?
------

* Voorkomen een klasse van bugs

.. code-block:: python

    a = 'tekst'
    b = True
    c = a / b


Waarom
======
dependent types?
----------------

* Type systeem kan veel meer bugs opvangen
* Deling door 0:

.. code-block:: agda

    _div_ : (dividend : ℕ)
            → (divisor : ℕ)
            → (proof : NonZero divisor)
            → ℕ


Waarom
======
dependent types?
----------------

* Postconditie van een functie:

.. code-block:: agda

    data List (A : Set) : Set where
      []  : List A
      _∷_ : A → List A → List A
    
    data ⊥X⊤ : Set where
      ⊤ ⊥ : ⊥X⊤
      ⟦_⟧ : X → ⊥X⊤

    data OList (l u : ⊥X⊤) : Set where
      Nil  : l ≤ u → OList l u
      Cons : ∀ x (xs : OList ⟦ x ⟧ u)
             → l ≤ ⟦ x ⟧ → OList l u


Waarom
======
dependent types?
----------------

.. code-block:: agda

    insert : ∀ {l u} x → OList l u
             → l ≤ ⟦ x ⟧ → ⟦ x ⟧ ≤ u
             → OList l u

    isort′ : List X → OList ⊥ ⊤
    isort′ = foldr (λ x xs → insert x xs (⊥ ≤ ⟦ x ⟧)
                                         (⟦ x ⟧ ≤ ⊤))
                   (Nil (⊥ ≤ ⊤))

    toList : ∀ {l u} → OList l u → List X

    isort : List X → List X
    isort xs = toList (isort′ xs)



Mijn thesis
===========

* Hoe verschillen talen met dependent types
* Universele technieken (views, universes)
* (Misschien) Technieken die taalspecifiek zijn
* Programming vs Extracting


Hoe verschillen talen met dependent types
=========================================
* Een vergelijking op basis van een aantal case studies
* Haskell, Agda, Coq, Idris
* vb. functors in Haskell en Agda


Functors
========
Haskell
-------

.. code-block:: haskell

    class  Functor f  where
        fmap :: (a -> b) -> f a -> f b

        (<$) :: a -> f b -> f a
        (<$) =  fmap . const

    (<$>) :: (Functor f) => (a -> b) -> f a -> f b  
    f <$> x = fmap f x



Functors
========
Agda
----

.. code-block:: agda

    record RawFunctor (F : Set → Set) : Set where
      field
        _<$>_ : ∀ {A B} → (A → B) → F A → F B

      _<$_ : ∀ {A B} → A → F B → F A
      x <$ y = const x <$> y


Universele technieken
=====================
views
-----

.. code-block:: agda

    data SnocView {A : Set } : List A → Set where
      Nil  : SnocView Nil
      Snoc : (xs : List A) → (x : A) →
        SnocView (append xs (Cons x Nil))

    view : {A : Set } → (xs : List A) → SnocView xs
    view Nil = Nil
    view (Cons x xs)                       with view xs
    view (Cons x .Nil)                    | Nil
      = Snoc Nil x
    view (Cons x .(append ys (Cons y Nil))) | Snoc ys y
      = Snoc (Cons x ys) y


Programming vs Extracting
=========================
* Sommige talen laten toe om uit een bewijs een programma af
  te leiden (Coq, Agda)
* Programma extraheren in een taal waarvoor er een sterke compiler bestaat


Referenties
===========

::

    Dependently Typed Programming in Agda
        http://www.cse.chalmers.se/~ulfn/papers/afp08
          /tutorial.pdf
    
    The Power of Pi
        http://cs.ru.nl/~wouters/Publications
          /ThePowerOfPi.pdf

    Agda by Example: Sorting
        http://mazzo.li/posts/AgdaSort.html
