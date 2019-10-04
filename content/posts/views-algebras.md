+++
title = "Views Algebras in Starling"
date = "2019-10-04T17:25:00+01:00"
author = "Matt Windsor"
tags = ["starling", "algebra"]
description = "In which I yammer on about the various forms of algebra that I explored in my PhD Starling work."
series = "Starling"
+++

In my PhD, I worked on Starling, a framework for building automated concurrent
verifiers _b/w_ [a tool](https://github.com/MattWindsor91/starling-tool) that
(informally) implements the framework.
This work drew much inspiration (and most of its soundness argument)
from the seminal
[Concurrent Views Framework](https://www.microsoft.com/en-us/research/wp-content/uploads/2016/02/views.pdf).
The most obvious location where this inspiration turns up is the fact that a
lot of the theory involves the manipulation of 'views algebras': abstract
combinations of sets of 'views' (bits of information about a shared state)
and operations that join and interpret them.

One of the
issues I've been grappling with since the
[2017 Starling paper](https://doi.org/10.1007/978-3-319-63387-9_27) is how we
take the algebraic shufflings the Starling tool performs and project them
into the meta-theory.  Ideally, each part of the Starling theory that involves
views algebras should depend on the most general algebraic varieties, and
use well-known varieties where possible.

I gave a talk on Starling on a research away trip a few weeks ago.  The next
talk namechecked the use of 'residuated monoids' to model part of a
system for modelling similarities between broadcast media.  (It seems like
these are [residuated lattices](https://en.wikipedia.org/wiki/Residuated_lattice), just without the lattice.)
This got me thinking: are the varieties from my thesis the right ones, and
what connections do they have to existing, but slightly obscure, varieties?

**Warning:** this post contains rambling about abstract algebras from someone
who doesn't have the necessary background to ramble coherently about abstract
algebras.

# The original algebras

These algebraic varieties correspond to those introduced in the original
paper, with one fairly pedantic change: I'll define all of the laws
over an equivalence relation {{< raw >}}\(\equiv\){{< /raw >}}, not equality.  This equivalence can be
anything at all, but in practice means 'refers to the same state set'.

A general rule of thumb is that, when we say 'views _XYZ_', we mean
'commutative _XYZ_ over {{< raw >}}\(\equiv\){{< /raw >}}'.

## Views semigroups

The _Views_ paper required only that views algebras form a
commutative semigroup.  This gives us a binary operator {{< raw >}}\(\bullet\){{< /raw >}} (or 'dot'), and the well-known laws:

- {{< raw >}}\(a \bullet b \equiv b \bullet a\){{< /raw >}} _(dot-commutativity)_
- {{< raw >}}\(a \bullet (b \bullet c) \equiv (a \bullet b) \bullet c\){{< /raw >}} _(dot-associativity)_

As we're defining the semigroup over an equivalence {{< raw >}}\(\equiv\){{< /raw >}}, we need an extra
bit of bureaucracy that tells us that we can use equivalences to rewrite both
sides of a {{< raw >}}\(\bullet\){{< /raw >}}:

- {{< raw >}}\((a \equiv b \wedge c \equiv d) \implies a \bullet c \equiv b \bullet d\){{< /raw >}} _(equiv-dot-congruence)_

## Views monoids

In practice, most assertion languages will have some idea of unit over
{{< raw >}}\(\bullet\){{< /raw >}}.
Having a unit turns out to be useful for two reasons: first, it lets us
represent the 'minimum knowledge' that we can stably hold about a proof
(a global invariant); second, it makes sequential reasoning a special case
of interference reasoning.

Adding a unit {{< raw >}}\(\epsilon\){{< /raw >}} gives us a
_views monoid_, and the laws:

- {{< raw >}}\(a \bullet \epsilon \equiv a\){{< /raw >}} _(dot-unit)_
- All laws of views semigroups

(We get {{< raw >}}\(\epsilon \bullet a \equiv a\){{< /raw >}} by commutativity.)

# Ordering and residuation

One of the big bits of theory that goes into the tool is the rearrangement of
the Views axiom soundness property (which resembles a proof rule à la
Owicki--Gries) into a form that's easy to automate.  Axiom soundness looks
like this:

{{< raw >}}
\[
\forall \{p\}\,c\,\{q\} \in \mathsf{Axiom} \ldotp
\;
[\![ c ]\!](\lfloor p \rfloor) \subseteq \lfloor q \rfloor
\;
\wedge
\;
\forall v \in \mathsf{View} \ldotp
[\![ c ]\!](\lfloor p \bullet v \rfloor) \subseteq \lfloor q \bullet v \rfloor
\]{{< /raw >}}

Part
of this involves subtracting the postcondition from both sides of the
interference check.  As the LHS of the check doesn't _have_ the postcondition,
we need to tweak views semigroups so that they:

1. have an operator that represents 'removing knowledge' from a view;
2. define (1) in the most general way possible (for now).

I'll get to (1) in a bit but, to set up (2), I'll take a detour and look at
adding ordering to views semigroups.  This pays off in letting us write laws
over inequalities rather than equivalences, and then later nudges us towards
the connection with residuated lattices.

## Ordered views semigroups

An ordering on views is an ordering on knowledge.  Usually, it'll be the inverse
of the inclusion ordering on the underlying state sets.  This means that view
ordering is partial: for example, 'the toaster is plugged in' and 'it's raining
in Paris' have no ordering relation.

Let's call this ordering operator
{{< raw >}}\(\sqsubseteq\){{< /raw >}}, and define it
in perhaps the most obvious way:

- {{< raw >}}\(a \sqsubseteq b \implies a \bullet c \sqsubseteq b \bullet c\){{< /raw >}} _(dot-increasing)_
- {{< raw >}}\(a \sqsubseteq a \bullet b\){{< /raw >}} _(dot-inflationary)_

(If we have a monoid, the first rule implies the second, by taking
{{< raw >}}\(a = \epsilon\){{< /raw >}}.)

We then add some laws that retrospectively force equivalence to become the
relation induced by taking
{{< raw >}}\(\sqsubseteq\){{< /raw >}}
in both directions.  (If we'd started with
the ordering first, we'd not need this, and just derive
{{< raw >}}\(\equiv\){{< /raw >}}
from {{< raw >}}\(\sqsubseteq\){{< /raw >}}.
But we didn't.)

- {{< raw >}}\(a \equiv b \implies a \sqsubseteq b\){{< /raw >}} _(inc-equiv)_
- {{< raw >}}\((a \sqsubseteq b \wedge b \sqsubseteq a) \implies a \equiv b\){{< /raw >}} _(inc-antisymmetry)_
- All properties of views semigroups

On views monoids, we can take
{{< raw >}}\(\epsilon \bullet a \equiv a\){{< /raw >}}
and
{{< raw >}}\(\epsilon \sqsubseteq \epsilon \equiv a\){{< /raw >}}
to derive the law
{{< raw >}}\(\epsilon \sqsubseteq a\){{< /raw >}},
which cements the unit as the minimum element of the ordering.

At this stage, you might wonder if there's a maximum element
(representing, for instance, the observation that matches no states).  While we
could add one, it won't play well with some of the later varieties in this
post, and working out how to rearrange them so that it will is left as an
exercise to the reader.

## Semi-residuated views semigroups

Let's now add in an operator that will let us express view subtraction.

I'll call this new operator 'sub' (which is a fairly inaccurate name, but is
the same number of letters and syllables as 'dot').
As a cheeky nod to separation logic, I'll use the notation
{{< raw >}}\(\bullet\!-\){{< /raw >}}; older Starling
works use
{{< raw >}}\(\setminus\){{< /raw >}}, which to me is a bit misleading.
(By swapping the parameters around, we get the famous
{{< raw >}}\(-\!\bullet\){{< /raw >}}, or 'magic wand',
of separation logic.  Through commutativity, we'll have that
{{< raw >}}\(\bullet\!-\){{< /raw >}} and
{{< raw >}}\(-\!\bullet\){{< /raw >}}
are the two residual operators once we get to fully residuated algebraic
structures.)

- {{< raw >}}\(a \sqsubseteq b \implies a \mathop{\bullet\!-} c \sqsubseteq b \mathop{\bullet\!-} c \){{< /raw >}} _(inc-sub-congruence)_
- {{< raw >}}\(a \sqsubseteq b \bullet c \implies a \mathop{\bullet\!-} b \sqsubseteq c\){{< /raw >}} _(sub-residual-forwards)_
- All properties of ordered views semigroups

I used to call these 'subtractive views semigroups', and this is probably
the name they'll appear under in the thesis.

### A rambling Starling detour

This definition looks rather bare: it only contains one side of the
residual property!  It turns out that most parts of Starling actually only need
this one side.  To see why, we look at the rewrites done to the axiom soundness
rule.  Both rewrites just change the part of the rule with the
{{< raw >}}\(\forall v\){{< /raw >}} quantification, and first is this:

{{< raw >}}
\[
\forall g \in \mathsf{View} \ldotp
[\![ c ]\!](\lfloor p \bullet  (g \mathop{\bullet\!-} q) \rfloor) \subseteq \lfloor g \rfloor
\]{{< /raw >}}

The intuition here is that, instead of quantifying over context views, we're
quantifying over _goal_ views: effectively, each
{{< raw >}}\(g\){{< /raw >}} stands for some
{{< raw >}}\(q \bullet v\){{< /raw >}} we might have considered in the original
formulation.  The important part for this post is that the sole appearance of
{{< raw >}}\(\bullet\!-\){{< /raw >}} is on one position of a subset relation.
For soundness, we need only show that each triple that satisfies the rewritten
rule also satisfies the axiom soundness rule, not the other direction.  The
combination of one direction of soundness plus one position of subtraction
means that only half of the residuation property need hold!

… Actually, we
do need an extra property ---
{{< raw >}}
  \(
    \forall p, q, v \ldotp
    \lfloor p \bullet v \rfloor
    \subseteq
    \lfloor p \bullet ((q \bullet v) \mathop{\bullet\!-} q) \rfloor
  \)
{{< /raw >}}
--- but this needn't actually be a property on the views algebra if the
reifier works in a certain way.


Incidentally, the _actual_ rule that most of the Starling work is based on
goes through a second rewrite:

{{< raw >}}
\[
\forall (g, \sigma_g) \in \mathsf{Definer} \ldotp
[\![ c ]\!](\lfloor p \bullet (g \mathop{\bullet\!-} q) \rfloor) \subseteq
\sigma_g
\]{{< /raw >}}

where
{{< raw >}}
\(
\lfloor v \rfloor =
\bigcap \left\{ \sigma_u | (u, \sigma_u) \in \mathsf{Definer} \wedge u \sqsubseteq v \right\}
\){{< /raw >}}, and
{{< raw >}}
\(\mathsf{Definer}\){{< /raw >}} is effectively a table of mappings from
'defining' views --- views that actually have some sort of meaning in the
shared state --- to their matching state sets.  This rewrite doesn't actually
need any further properties other than semi-residuation and extra property
above, though.

# The twilight zone

Certain specific instantiations of Starling need more structure from the
views algebra.  When I was writing the thesis, I thought that I'd be able to
come up with a general algebra that captured all the things that those
instantiations need without over-restricting.  This led to 'separating' views
semigroups, which I'll discuss in a bit.  'Separating' semigroups are a bit
of a hack.

First though: I mentioned residuated monoids a while back --- where would these
fit in the grand scheme of views algebras?

## Residuated views semigroups?

If a semi-residuated views semigroup has half of the residuation property,
then presumably a _residuated_ views semigroup would have the whole residuation
property:

- {{< raw >}}\(a \mathop{\bullet\!-} b \sqsubseteq c \implies a \sqsubseteq b \bullet c\){{< /raw >}} _(sub-residual-backwards)_
- All properties of semi-residuated views semigroups

I don't know of anything Starling-related that uses _just_ this property, so
I'll move on.  (Of course, a residuated views monoid is just a residuated
views semigroup that is also a residuated views monoid.)

## Cancellative views semigroups?

A rather famous subclass of views semigroups is _separation algebras_
(partial cancellative commutative monoids).  Here, cancellation is useful for
expressing the ability to attach and detach arbitrary view 'contexts', and
usually comes up in the context of reasoning about the _frame rule_, which,
in separation logics, looks like this:
{{< raw >}}
\[
\vdash \{p\}\;C\;\{q\} \implies
\forall v \ldotp \vdash \{p \bullet v\}\;C\;\{q \bullet v\}
\]
{{< /raw >}}

One way to add cancellativity to residuated views semigroups, which is the
way that appears (for better or worse) in the thesis, involves adding one law:

- {{< raw >}}\(
  \forall a, b \ldotp
   a \sqsubseteq (a \bullet b) \mathop{\bullet\!-} b
  \){{< /raw >}} _(cancel-backwards)_
- All properties of residuated views semigroups

From this law, we can derive something that looks more like the usual
cancellativity property
({{< raw >}}\(
   a \bullet c \sqsubseteq b \bullet c \implies
   a \sqsubseteq c
  \){{< /raw >}}) through a large amount of residuation rewriting.  By then
applying the relationship between ordering and equivalence, we get
'proper' cancellativity
({{< raw >}}\(
   a \bullet c \equiv b \bullet c \implies
   a \equiv c
  \){{< /raw >}}).

A quick `ripgrep` of the Starling Coq tree suggests that nothing actually _uses_
this property, at least directly.  As such, the jury is still out as to whether
it actually buys us anything, or whether it could be used to clear up some of
the mess to come in the next variety.  As there are similarities between
this form of cancellativity and the extra property mentioned above, I suspect
there was a connection between the two once.

## 'Separating' views semigroups

Ok, this is where things get messy.  Later parts of the Starling work assume
that we can decidably rewrite any view (or, more correctly, an expression tree
representing the view) so that the view is in something called _list-normalised
form_: it's a chain
{{< raw >}}
\(
a \bullet b \bullet \dotsb \bullet \epsilon
\)
{{< /raw >}} with absolutely _no_ instances of
{{< raw >}}
\(
\mathop{\bullet\!-}
\)
{{< /raw >}}
in it (the
{{< raw >}}
\(
\mathop{\bullet}
\)
{{< /raw >}}ed-together views are then called _view atoms_
).  This makes implementing
{{< raw >}}
\(
\mathsf{Definer}
\)
{{< /raw >}} in a decidable, pattern-match based form easier, as we can just
check lists of atoms against each other.

Of course, any decision process for rewriting
{{< raw >}}
\(
\mathop{\bullet\!-}
\)
{{< /raw >}}s depends a _lot_ on the semantics of
{{< raw >}}
\(
\mathop{\bullet\!-}
\)
{{< /raw >}}, and the existing classes don't say much of use.  One of the rules
that we could do with is something like
{{< raw >}}
\(
a
\mathop{\bullet\!-}
(b \bullet c)
\;=\;
(a
\mathop{\bullet\!-}
b) \mathop{\bullet\!-} c
\)
{{< /raw >}};
then, we can reduce an arbitrary subtraction into iterated subtractions of
one atom each.  Then, we need some way of recursively moving that subtraction
down the atoms of a view: some rule
{{< raw >}}
\(
(a \bullet b)
\mathop{\bullet\!-}
c
\;=\;
(a
\mathop{\bullet\!-}
c)
\bullet
(a
\mathop{\bullet\!-}
R)
\)
{{< /raw >}} where
{{< raw >}}\(R\){{< /raw >}} is some _remainder_ of
{{< raw >}}\(c\){{< /raw >}} left over from the first subtraction.
Then, we just need some model for subtracting atoms from atoms --- this depends
wildly on what the atoms _are_, and various forms exist in the thesis!

At this stage of Starling's development, we had a tool, and the tool handled
multisets (the free commutative monoid!).  As a result, its decision process
handled subtraction over multisets.  When I came to formalise this in the
thesis, I took things I knew about multisets, tried to find the smallest
covering set of algebraic properties that let the decision process work, and
turned them into a variety.  This is the 'separating' views semigroup, where:

- {{< raw >}}\(a \mathop{\bullet\!-} b \sqsubseteq c \implies a \sqsubseteq b \bullet c\){{< /raw >}} _(sub-residual-forwards)_
- {{< raw >}}\((a \bullet b) \mathop{\bullet\!-} c \equiv (a \mathop{\bullet\!-} c) \bullet (b \mathop{\bullet\!-} (c \mathop{\bullet\!-} a))\){{< /raw >}} _(sub-of-dot)_
- All properties of cancellative views semigroups

Other properties, such as the 'sub by dot' property mentioned above, fall out
of these and previous laws.

The _sub-of-dot_ property might be familiar to anyone who has squinted for too
long at the 'truncated subtraction' operator on natural numbers.  This property
neatly solves the 'what is the remainder?' problem for rewriting
{{< raw >}}\(\mathop{\bullet\!-}\){{< /raw >}}; unfortunately, it does so
by seemingly restricting the definition of the operator so much that I've only
been able to instantiate four types of 'separating' semigroup:

- singletons;
- natural numbers;
- pointwise lifts, and other such transformations, on existing separating
  semigroups (one of which, the functional lifting of natural numbers, is
  precisely multisets!);
- abstract expression trees that compile to 'concrete' views.

Indeed, it may well be that all of these varieties are just isomorphic to
multisets, which is... depressing, to say the least.  However, multisets have
proven to be a versatile views model in the past 5 years, so maybe it isn't all
that bad :)

### As residuated lattices

One interesting corollary of the accidental restriction of 'separating'
semigroups to naturals and constructs over naturals is that they form
lattices (and, so, residuated lattices).  The construction is straightforward:
join is 'maximum'
({{< raw >}}
\(
a \bullet (b
\mathop{\bullet\!-}
a
\)
{{< /raw >}}) and meet is 'minimum'
{{< raw >}}
\(
a \mathop{\bullet\!-} (a
\mathop{\bullet\!-}
b)
\)
{{< /raw >}}.  Proofs left as an exercise to the reader.

It'd be nice (by which I mean it'd give me a tingly, warm sensation) if
most, or all, useful views algebras formed residuated lattices.  As of yet,
I haven't actually thought of anything that this gives us (though some of the
usual lattice properties about the join operation appear in some of the Coq
mechanisation for list normalisation, so... maybe?)

# Conclusion

The idea that we can progressively transform parts of existing meta-theory
(eg the _Views Framework_) into automatable proof rules by progressively adding
more algebraic structure was, and is, something that massively appeals to me
as a functional-programmer-in-exile.  Certainly, adding ordering and a weak
form of 'view subtraction' seems to work quite well in practice.

However, a spectre is haunting the world of views algebra varieties that I
explored in my PhD --- the spectre of 'subtractive' views semigroups.  It's
becoming increasingly clear to me that these are a red herring, a wrong turn in
the otherwise quite neat and tidy progression of varieties.  What isn't clear
yet is what, if anything, should replace them.

## Open problems

These aren't necessarily 'challenging' problems --- just ones for which I
haven't been able to allocate enough brain space.

- Is the 'separating' variety inhabited by anything that isn't isomorphic to
  multisets?
- Are there any interesting algebraic varieties that add structure to
  semi-residuated semigroups, but are more elegant than the progression shown
  above?
- Does cancellativity give us anything?  I feel like it should, as it's a key
  ingredient of real-world views algebras, but it seems to have fallen behind
  the sofa in Starling.
- Are there any useful generalisations of the multiset-specific subtraction
  decision process, and do they lead to interesting algebraic varieties?
- Is every (semi-residuated, residuated, cancellative, _etc._) semigroup a
  monoid (eg, do the laws imply the existence of a unique unit)?
- Is there any way to extend these varieties with a notion of a
  'largest' view (representing, for example, an assertion that no states can
  satisfy), without collapsing the whole variety down to a singleton carrier?
  (Several of the laws defined above seem to rule this out.)
