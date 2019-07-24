---
layout: post
title:  Functional Programming GUIs (part 2) - Arrowized FRP
---

Functional Reactive Programming (FRP) is a declarative method of creating systems that react to changes of the environment over time. The initial work of \citeauthor{Elliott2005} introduces two type of inputs to the system as values that change over time (time varying values): behaviors and events. Behaviors are continuous varying values:

<p align="center"><img src="/_posts/tex/c28bb3d719f344598bb7f9d1a37db92e.svg?invert_in_darkmode&sanitize=true" align=middle width=182.93033714999999pt height=11.4155283pt/></p>
\\

while events are discrete values:

\\
<p align="center"><img src="/_posts/tex/3f6e8d5c093e4478a588b5e218a8ac14.svg?invert_in_darkmode&sanitize=true" align=middle width=162.5107407pt height=16.438356pt/></p>
 
The first type of value helps modeling physical systems for instance, where phenomena like position, velocity, acceleration depend on continuous input while events help model situations with discrete inputs, like GUI interactions with a user. The focus of the first formulation of FRP was animation (the original implementation is called FRAN - functional reactive animation) and this is the reason for this work to be focused on continuous values.

The original implementation is very expressive and flexible, posing no significant difficulties in understanding or implementation while being composable. However, it allowed accumulation of events in the event system due to the lazy nature of the host implementation language - Haskell - which only evaluates a value when it is explicitly requested by the effect system. This could potentially lead to memory exhaustion causing a problem called *space leaks*. When the events are finally evaluated they create an execution bottleneck that leads to longer execution time known as a \emph{time leak} or a \emph{global delay}. For the continuous counterpart it required frequent sampling of the input space that triggered extraneous re-renderings of the entire system for no modifications or only partial modifications of the system state, a problem known as \emph{unnecessary update}. The implementation also allowed defining behaviors that depended on past values (causing memory leaks) or future values (breaking causality, thus not implementable).

Subsequent updates to the classical FRP utilize the observation that the discrete case of the event system is isomorphic to the continuous case for optional values:

<p align="center"><img src="/_posts/tex/0c6c3406246091204f2b27a2a73685a8.svg?invert_in_darkmode&sanitize=true" align=middle width=464.01133350000003pt height=16.438356pt/></p>

leading to both events and behaviors being represented by a \emph{signal}:

<p align="center"><img src="/_posts/tex/1ca44d8d5258aaf018486446e2010ffb.svg?invert_in_darkmode&sanitize=true" align=middle width=162.66579449999998pt height=14.611878599999999pt/></p>

With this new data type it is possible to introduce a more restrictive language as a reactive front-end with a more flexible compilation target language that allows to limit the problem surface for the space and time leaks, as well as for the unnecessary updates.

These approaches, while working, lose some of the expresiveness of the classical FRP. The efforts to reclaim this expresiveness while maintaining the safety guarantees lead to \emph{arrowized FRP}.

An *arrow* is the generalization of a function at type level in a pure functional programming language. In a category theory interpretation of arrows that suits computing best (where \(V\) is \(Set\)) they are monoids in the category of bifunctors \( C^{op} \times C \rightarrow V \)where \(V\) is a symmetric closed monoid and \(C\) is \(V\)-enriched\cite{Uustalu2008}. This allows arrows to compose while retaining some context of the computation in the chaining.

*Arrowized FRP* overcomes the limitations of classical, real-time and event-based FRP by restricting access to signals to avoid time and space leaks and by reclaiming the expressiveness of classical FRP through the properties of an arrow. It defines a datataype

<p align="center"><img src="/_posts/tex/721a2ca8443c4c937739a78683095892.svg?invert_in_darkmode&sanitize=true" align=middle width=232.58207445pt height=14.611878599999999pt/></p>

that abides by the arrow laws. Because the signal types available to the programmer are specified at the source level and their constructors are not available for direct usage the usage of signals is both restricted to avoid leaks and respect causality. Even with these measures, leaks are still possible in the host implementation due to the \emph{instant update assumption}\cite{Czaplicki2012} that states that the semantics of arrows can be respected as long as sampling interval for updates goes to 0.

*Czaplicki* uses the concept of signal arrows in the creation of his Elm language to disallow folds from the past of signals that drop signals during the fold. For this reason the type representing a signal in Elm is only an applicative functor, not a monad (join-ing would be flattening the signal). At the same time, a signal is isomorphic to an arrow from the World (environment) to the value \(\alpha\):

<p align="center"><img src="/_posts/tex/a527422fc394efb6b9c5c73fac84ea8b.svg?invert_in_darkmode&sanitize=true" align=middle width=179.72918205pt height=14.611878599999999pt/></p>

This interpretation gives the backend compiler the flexibility to use the arrow interface to interpret the reactive interface (*foldp* and *lift*).

Citing difficulties in adoption of the language (A farewell to FRP, Czaplicki, E., May 10 2016, https://elm-lang.org/blog/farewell-to-frp) Czaplicki2012 moves Elm with version 0.17 away from FRP in 2016, favoring a message-passing concurrent approach similar to Erlang.

While arrowized FRP presents good compositional properties when representing signals as arrows a practical implementation that does not leak time and memory will impose significant limits on these properties. Arrows impose a point-free style that is notoriously difficult for programmers. Defining lambda calculus in terms of arrows to allow decoupling of computations implies adding separate arrow interface functions for function application, choice and feedback and making concurrent arrows naturally flows towards the Erlang actor model. Arrows compose using arrow transformers (a generalization of monad transformers), but these structures imply unwrapping the nesting levels to perform the desired effect and consume the effect stack. However, some arrow based combinations might still yield good results as a means to pass along application state in an effect composition chain.
