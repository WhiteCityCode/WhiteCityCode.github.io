---
layout: post
title:  Functional Programming GUIs (part 1)
---

Programming GUIs poses a number of interesting problems and has known over the years a number of interesting solutions both in the imperative programming paradigm and in the functional one.

For one it is considered unacceptable for a graphical interface to block while some work is being performed due to usability reasons and thus entails the existence of at least 2 threads of computing, one for background work and one for interface updates. Serializing the actions and their results in a consistent and predictable manner has been a challenge for programmers for various reasons, ranging from complexity of the state and state update dependency calculations to displaying proper results to the user.

In recent years versions of a functional approach to the problem -- namely Functional Reactive Programming -- have known a great deal of success being implemented in imperative languages mainly due to providing a good structure to Single Page Applications written in javascript. This is an indication that there is a great interest in the simplicity provided by the functional framework to the overall process of structuring an application.
<!--more-->

As popular as the approach is it still does not fulfill the age-old promise of perfect decoupling of components in programming nor the one of minimizing the cognitive burden on the programmer when trying to achieve this decoupling. Moreover, complexity appears to be the single most frequent cause of software errors, leading to less stable software. Due to concurrency this issue is especially prevalent in GUI programming.

The scope of the present inquiry is thus multi-folded. We want to find a solution that presents these characteristics:

- it is expressive enough so it can model a GUI program with all its aspects
- it is composable such that larger programs can be constructed from smaller GUI components (thus having a functional basis)
- it restricts complexity to the component being implemented, thus reducing programmer cognitive burden
- it is type-safe, reducing to the minimum the runtime faults
- it has at least moderate performance characteristics for today's standards

As secondary objective we also aim to implement the chosen solution in a proof-of-concept application that will allow us to evaluate how ergonomic is the proposed solution.

We compare three selected approaches to this problem and we analyze them with respect to the research objective. 

Arrows are generalization of morphisms with a container component such that decidability is easily implemented based on the contained value. Arrowized FRP composes well, is expressive and type-safe but has known problems with space and time leaks.

Algebraic event handlers are a recent approach to handling composition in the presence of effects. Effects in a program form a free monad that can be folded using an algebra (the free monad's catamorphism). The program is a tree that is folded to the program's result. As for the composition of effects and handlers, effects can be tagged in a composed effect and can be folded based on the tag. Effect handlers look very promising but there are questions with regard to both composability and performance.

And lastly monad-comonad pairings act as pairs of space-movement to describe computations in a GUI program. The pairing of two functors expresses the connection between the two contexts of the functors that allow them to cancel each other out in the process of interpretation to obtain a context-free value. In the case of a comonad-monad pairing the monad can peel off one layer of comonad context that it consumes to move inside the space defined by the comonad. The objections to the approach are, just as in the case of effect handlers related to performance and composition. Comonads use Comonad transformers to compose effects and transformer stacks have problems with the nesting of effect stacks. 
