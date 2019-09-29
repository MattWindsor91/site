+++
title = "Views Algebras in Starling"
date = "2019-09-21T19:32:00+01:00"
author = "Matt Windsor"
tags = ["starling", "algebra"]
description = "In which I describe the various forms of algebra that I explored in my PhD Starling work."
series = "Starling"
draft = true
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

I gave a talk on Starling on a research away trip last Wednesday.  The next
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

- {{< raw >}}\(a \bullet b \equiv b \bullet a\){{< /raw >}} (dot-commutativity)
- {{< raw >}}\(a \bullet (b \bullet c) \equiv (a \bullet b) \bullet c\){{< /raw >}} (dot-associativity)

As we're defining the semigroup over an equivalence {{< raw >}}\(\equiv\){{< /raw >}}, we need an extra
bit of bureaucracy that tells us that we can use equivalences to rewrite both
sides of a {{< raw >}}\(\bullet\){{< /raw >}}:

- {{< raw >}}\((a \equiv b \wedge c \equiv d) \implies a \bullet c \equiv b \bullet d\){{< /raw >}} (equiv-dot-congruence)

## Views monoids

In practice, most assertion languages will have some idea of unit over
{{< raw >}}\(\bullet\){{< /raw >}}.
Having a unit turns out to be useful for two reasons: first, it lets us
represent the 'minimum knowledge' that we can stably hold about a proof
(a global invariant); second, it makes sequential reasoning a special case
of interference reasoning.

Adding a unit {{< raw >}}\(\epsilon\){{< /raw >}} gives us a
_views monoid_, and this law:

- {{< raw >}}\(a \bullet \epsilon \equiv a\){{< /raw >}} (dot-unit)

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

- {{< raw >}}\(a \sqsubseteq b \implies a \bullet c \sqsubseteq b \bullet c\){{< /raw >}} (dot-increasing)
- {{< raw >}}\(a \sqsubseteq a \bullet b\){{< /raw >}} (dot-inflationary)

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

- {{< raw >}}\(a \equiv b \implies a \sqsubseteq b\){{< /raw >}} (inc-equiv)
- {{< raw >}}\((a \sqsubseteq b \wedge b \sqsubseteq a) \implies a \equiv b\){{< /raw >}} (inc-antisymmetry)

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

- {{< raw >}}\(a \sqsubseteq b \implies a \mathop{\bullet\!-} c \sqsubseteq b \mathop{\bullet\!-} c \){{< /raw >}} (inc-sub-congruence)
- {{< raw >}}\(a \sqsubseteq b \bullet c \implies a \mathop{\bullet\!-} b \sqsubseteq c\){{< /raw >}} (sub-residual-forwards)

This definition looks rather bare: it only contains one side of the
residual property

I used to call these 'subtractive views semigroups', and this is probably
the name they'll appear under in the thesis.

# The twilight zone

## Residuated views semigroups


- {{< raw >}}\(a \mathop{\bullet\!-} b \sqsubseteq c \implies a \sqsubseteq b \bullet c\){{< /raw >}} (sub-residual-backwards)

## 'Separating' views semigroups

### As residuated lattices

# Conclusion
