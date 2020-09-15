---
layout: post
title:  Learnings from Hints and Principles for Computer System Design
date:   2020-09-14 00:00:00 -0700
categories:
tags:
---

Hints and Principles for Computer System Design by Butler Lampson.

# Principles

## 1. Abstraction - Have a spec

The idea is to have a specification for the computer system that tells its clients
− what: everything they need to know to use the system,
− but **not** how: nothing about how it works inside—the code.

## 2. Writing a spec—KISS: Keep It Simple, Stupid.

## 3. Writing the code: Correctness — Get it right

Why are static types good? For the same reason that static checking in general is good: the compiler can try to prove theorems about your program, and if it fails you have found a bug early, when it’s cheap to fix

## 4. Modules and interfaces — Keep it clean. Keep basic interfaces stable.

A really successful interface is like an hourglass: the spec is the narrow neck, with many clients above and many codes below; it can live for decades. 

### Reuse of Components
A module that is engineered to be reused in several systems is called a component. Obviously it’s better to find a component that does what you need than to build it yourself (don’t reinvent the wheel), but there are some pitfalls:

− You need to understand its spec, including its performance.

− You need to be confident that its code actually satisfies the spec and will be maintained.

− If it doesn’t quite do everything that you want, you have to fill in the gaps.

− Your environment must satisfy the assumptions the component makes: how it allocates
resources, how it handles exceptions, how it’s configured, and the interfaces it depends on. 

There are two ways to keep from falling into one of these pitfalls:

• Copy and paste the module’s code into your system and make whatever changes you find necessary. This is usually the right thing to do for a small component, because it avoids the problems listed above. The drawback is that it’s hard to keep up with bug fixes or improvements.

• Stick to the very large components usually called platforms. There will only be a few of them to learn about, they encapsulate a lot of hard engineering work, and they stay around for a long time because they have a viable business model (since it’s impractical to write your own).R39 A well-maintained library can also be a source of safe components.

### Open system - Don’t hide power. Leave it to the client.

The point of an abstraction is to hide how the code is doing its work, but it shouldn’t prevent a client from using all the power of its host. 

## Point of View

A good way of thinking about a system makes things easier, just as the center-of-mass coordinate system makes dynamics problems easier. It’s not that one viewpoint is more correct than another, but that it’s more convenient for some purpose.

(Naming & Syntax)

# Goals and Techniques

## Goals - STEADY

### 1. Simple: Is it clean? Frugal. Beauty.

Keywords: abstraction, action, extensible, interface, predictable, relation, spec.

### 2. Timely: Is it ready? First. Time to market.

### 3. Efficient: Is it fast? Fast. Economy.

Keywords: abstraction, action, extensible, interface, predictable, relation, spec.

There are two basic ways to reduce latency: concurrency and fast path—do the common case fast, leaving the rare cases to be slow. 

Almost the opposite of a fast path is a bottleneck, the part of the system that consumes the most time. Look for the bottleneck first.

Predictable performance: Don’t try to be precise; that’s too hard. It’s enough to know how to avoid disaster, as in paging, where you just need to keep the working set small enough.

It often pays to compress data so that it’s cheaper to store or transmit. The most powerful compression produces a summary that is much smaller than the input.

*Locks* don’t work well in a distributed system because they don’t play nice with partial failures. *Leases* can be a workaround. The only meaningful content in an asynchronous message is facts that are stable: once they are true, they are true forever. For example, a lease implies “𝑃 holds until time 𝑡,” which is stable. “𝑃 holds until 𝑄” might be stable too, but a failure can make 𝑄 inacces- sible. A fact from an eventually consistent system is stable only if it has been synced.

### 4. Adaptable: Can it evolve? Flexible. Evolution.

Keywords: dynamic, index, indirect, scale, virtualize.

Scaling requires:

- Modularity for algorithms, so it’s easy to change to one that scales better.
24

- Automating everything, both fault tolerance and operations, so that a human never touches just one machine (except to replace it if the hardware fails).

- Concurrency that scales with the load by sharding: different shards are independent be- cause they don’t share variables or resources: all communication is asynchronous.

### 5. Dependable: Does it work? Faithful. Fidelity.

Keywords: atomic, consensus, eventual, redundant, replicate, retry.

On Security:
− Focus: figure out what you really need to protect.
− Lower aspirations: secure only things so important that you’ll tolerate the inconvenience.
− Isolation: sanitize outside stuff to keep it from hurting you, or don’t share dangerous stuff.
− Whitelisting: decide what you do trust, rather than blacklisting what you don’t.

### 6. Yummy: Will it sell? Fashionable. Elegance.

Keywords: becoming, indirect, interface, recursive, tree.

## Techniques - AID

- Approximate: rather than exact, perfect or optimal results are almost always good enough, and
often much easier and cheaper to achieve. Loose rather than tight specs are more likely to be
satisfied, especially when there are failures or changes. Lazy or speculative execution helps to
match resources with needs.

- Incremental: design has many aspects; often they begin with “i”. The most important is to
build the system out of independent, isolated parts called modules with interfaces, that you can put together in different ways. Such parts are easier to get right, evolve and secure, and with indirection and virtualization you can reuse them in many different environments. Iterating the design rather than deciding everything up front keeps you from getting too far out of touch with customers, and extensibility makes it easy for the system to evolve.

- Divide and conquer: the most important idea, especially in the form of abstractions with clean specs for imposing structure on your system. This is the only way to maintain control when the system gets too big for one person’s head, now or later. Other aspects: making your system concurrent to exploit your hardware, redundant to handle failures, and recursive to re- use your work. The incremental techniques are other aspects of divide and conquer.

# Process

ART: 

- Architecture:

Design that really gets done, and documented so that everyone can learn it.

- Automation:

Code analysis tools (very cheap for the errors they can catch) and build tools.

- Review

Design review — manual, but a much cheaper way to catch errors than testing.

Code review — manual, but still cheaper than testing.


- Techniques and Testing

Unit and component tests; stress and performance tests; end-to-end scenarios.

# Oppositions

Systems are complicated because it’s hard work to make them simple, and because people want them to do many different things.

Still, it’s best to add features and generality slowly, because:

- You’re assuming that you know the customers’ long-term needs, and you’re probably
wrong. It’s hard enough to learn and meet their immediate needs.

- It takes time to get it right, but once it’s shipped legacy customers make it hard to change.

- More features mean more to test, and more for a bad guy to attack.