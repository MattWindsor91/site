---
title: "Side Projects in 2020"
date: 2020-01-05T23:32:00Z
description: "In which I discuss various projects I'd like to do 'on the side'
of my current work in 2020."
dropCap: true
displayInMenu: false
displayInList: true
draft: false
---

It's a new year, and the night before I go back down the mines of
[automated compiler testing](https://github.com/MattWindsor91/act).  It's also
a while since I've posted here, so I thought I'd fix that.  But what
to talk about?

It crossed my mind that I have a lot of computer-_ish_ things I
want to do this year that aren't directly related to my research remit.  I
figured it might be interesting to discuss them for several reasons:

- It gives me something to look back on in 2021 to see how many
  diverse things I managed to do in the year;
- It lets me elaborate on some of the thoughts and justifications swimming
  in my head;
- If any of them seem daft, or have already been done, etc., someone might stop me! :)

# A more practical _Starling_ tool

My PhD focused on _Starling_, an approach for building concurrency verification
tools on top of sequential ones.  A large amount of the PhD focused on building
[one such tool](https://github.com/MattWindsor91/starling-tool) in F#.  This
tool was mainly a proof of concept, and has a few issues:

- The language is a strange mashup of C, pseudocode, and high-level
  concepts.  I found during the PhD that the focus of the language moved from
  being 'close to the metal' and nominally intended for concurrency
  implementers to being more abstract and intended for sketching and proving at
  the algorithm level.  A more useful language would be closer to pseudocode.
- The tool's implementation didn't scale well as more and more concepts, and
  more and more backends, were added to it.  A re-thinking of the tool might
  make it easier both to handle the existing burden and to expand it later on.
- Specific issues with the present tool include the way in which parsing and
  desugaring interoperate, as well as the tool's poor error reporting (caused by
  not enough information being propagated or inferred throughout the process).

Why would a new tool be useful, given that the likelihood of actual adoption is
unlikely?  Aside from my own personal amusement, as well as possibly being a
springboard for further _Starling_ research, one thought that came to me is the
possible use of such a tool as an educational aid (for teaching the joys of
concurrent proof, perhaps!).  This would require it to be
both robust and forgiving, of course.

I've toyed with the idea of making a new tool in C# with a language based on
Pascal (name ideas including 'Pascal with Views and Concurrency', or PVC for
short!), but not made much progress -- this would be a huge undertaking for
little immediate gain, to be sure.


# A relational local _Views Framework_

The
[theory side of _Starling_](https://gitlab.com/MattWindsor91/starling-coq)
makes much use of the
[_Views_ framework](http://microsoft.com/en-us/research/wp-content/uploads/2016/02/views.pdf),
both directly and in altered form (to add support for local variables).  While
these forms suffice for _Starling_, the way in which I added local variables on
top of _Views_ was inelegant, and the limitation of views to single-state
facts (rather than abstracting over possible transition relations) came up in my
viva.

Something I've been thinking about, and made a little bit of progress towards in
Coq (watch this space!), is experimenting with a form of _Views_ in which views
reify into relations over both local and shared states.  At this stage, I'm
unsure as to whether this is possible (though I know that support of local
states by encoding into functions from such state to view _is_, as are
concurrency reasoning systems over relations).

At this stage, this is merely an excuse for me to use Coq in my downtime (I'm
one of those people who enjoys using a theorem prover to unwind :D): it
might not actually lead anywhere; and, given the direction of concurrency proof,
the impact will likely be low.  But I want to keep exercising my concurrency
reasoning muscles over the immediate future, and this seems like a relatively
gentle way in which to do so.


# _Travesty_, but for applicative functors

[_Travesty_](https://github.com/MattWindsor91/travesty) is an OCaml library for
defining schemes of monadic traversal, bifunctors, bi-traversals, and other
'map over data structures in complicated ways' happenings.  It spun out of
_ACT_, the automated compiler testing environment on which I work.

At present, _Travesty_ only supports generation of traversals and bi-traversals
over monads.  This is because monads, at the time, seemed like they were easier
to understand in OCaml, and the vast majority of use cases for _Travesty_ are
indeed on monads (almost entirely the `Or_error` monad!).  However, I'm aware of
at least one person on the OCaml forums who has expressed interest in _some_
library for generating traversals over applicative functors.

Re-purposing _Travesty_ to accept applicatives will be a fairly nasty breaking
change, as all of the dependent code's instantiations of _Travesty_'s functors
will need to re-target over applicatives.  However, it feels like this should
be done sooner rather than later, in the rare event that said dependencies grow
to become more than just _ACT_.

I have a few other features I'd like to add to _Travesty_ eventually, such as
PPX support for auto-deriving traversals in certain cases... but I've never been
able to justify doing them in work time, and free time has gone to other things.

As well as _Travesty_, I've been contemplating splitting other bits of ACT off
into separate libraries.  For instance, there is a fairly self-contained
subsystem of ACT that deals with command plumbing: building Unix-like 'filter'
programs; remote invocation over `ssh` and `scp`; and other miscellanea.  I've
been wanting to peel it off for a while, but haven't quite worked out the
right niche for it.

# Modernised radio playout system

Finally, the thing that's been taking up most of my free time is an unofficial
['modernisation'](https://github.com/MattWindsor91/BAPS2) of _BAPS_, the playout
system used by my alma mater's [university radio station](https://ury.org.uk).
The original _BAPS2_ was written in C++/CLI, is tied to the .NET Framework and
Windows Forms, and could be easier to maintain if modernised and restructured
somewhat.

I've made a fair bit of progress in porting bits of BAPS2 to C# on .NET Core,
recreating the presenter frontend in WPF, and trying to make a more easily
debuggable server, but there is a _lot_ of work to do, both to achieve feature
parity and to make sure the result is actually maintainable.  Some of the code
I've produced so far is in need of hammering down for simplicity and
scrutability - something that I really need to practice more often!

# Other things I want to do

These read like a list of new years' resolutions, but I'm not committing to them
in such terms, so... we'll see.

- Hit the gym (I've had a membership since around April, but started flagging on
  attendance towards the end of the year);
- Resume learning to drive (thwarted somewhat by semi-moving to London);
- Become active again in the labour movement (I go through spurts of enthusiasm
  followed by apathy, but now is likely to be the best time to re-enthuse
  myself);
- Contribute to at least one free/open-source project that isn't one of my own;
- Try to rekindle my interests in music making and game making.
