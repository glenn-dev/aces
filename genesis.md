# Aces — Genesis

**First written:** 2026-09-03

## Origin

Aces begins with a simple idea: build a modern game inspired by the fundamental concepts that made *Advance Wars* memorable.

It is not intended to be a faithful recreation.

*Advance Wars* is the starting point: a bounded environment in which players, teams, entities, resources, actions, territory, and events interact according to explicit rules and toward defined objectives.

Aces will use that foundation to develop its own systems, identity, and design.

## Vision

The first goal is to build a game.

A small, understandable, and playable system should come before a generalized platform.

At its core, Aces is interested in environments where participants act on entities under explicit rules, and where those actions produce observable consequences that change the state of the environment.

The game is the first concrete domain in which to explore that idea.

The game should be designed around a logical core that is as independent as practical from its presentation. Names, images, animations, sounds, terminology, entity types, and potentially some attributes should be configurable without requiring the fundamental interaction model to be redesigned.

This creates the possibility for Aces to support multiple thematic or visual implementations over the same underlying foundations.

## A Game First, an Experiment Second

Aces is also an experiment.

A game provides a controlled environment in which entities can be created, assigned capabilities and resources, directed toward other entities, affected by events, and transformed through actions whose consequences alter the state of the world.

The project will explore whether some abstractions that emerge from building such a system can later be reused outside the game itself.

At a very high level, a flow such as:

> entity → target → action → consequence → new state

may describe both a game unit acting on another entity and a software entity — such as a document, message, or piece of code — interacting with another system, API, tool, agent, or route.

This is a hypothesis to explore, not a requirement that the initial game must satisfy.

Reusable abstractions should be extracted only after they prove useful in the concrete implementation.

## Configurability and White-Label Ambition

Aces should separate its underlying logic from the way that logic is presented to the user whenever doing so provides real value.

The initial game may use a military or geopolitical theme, but the underlying concepts should not be unnecessarily coupled to that vocabulary or presentation.

Persistent definitions of relevant domain entities may produce ephemeral runtime instances within a game or scenario.

The vocabulary and exact domain model should evolve through game design rather than being fixed by this document.

This configurability is the current meaning of the project's white-label ambition. The exact boundaries of that capability are intentionally left open and should emerge through implementation rather than premature abstraction.

## AI-Native and Human-in-the-Loop

Aces should be developed as an AI-native project.

AI should not be treated merely as a feature added after the core product exists. The project should explore how humans, software agents, deterministic systems, and tools can cooperate in environments with explicit state, permissions, actions, constraints, events, and consequences.

Human agency should remain visible and intentional.

The project should favor workflows in which important state and actions can be inspected, understood, and, where appropriate, approved or redirected by a human participant.

## Core Principles

- Start with the game.
- Prefer concrete systems before generalized abstractions.
- Keep domain logic decoupled from presentation where it provides meaningful flexibility.
- Make important state, actions, constraints, and consequences explicit and inspectable.
- Design for iteration rather than premature completeness.
- Treat documentation as part of the product.
- Use AI as a participant in the development process, not as a substitute for understanding the system.
- Preserve human agency in workflows involving automation or intelligent agents.
- Extract reusable models only after they prove valuable in the concrete game.
- Let implementation refine the vocabulary and abstractions instead of forcing the implementation to fit an early theory.

## Initial Scope

The first concrete expression of Aces will be a playable strategy game.

It should begin deliberately small, using a bounded world, explicit actors, state, actions, resources, events, constraints, and objectives.

Detailed mechanics and domain vocabulary belong to the evolving game-design documentation rather than to this Genesis.

## Boundaries

Aces does not begin as a generalized workflow engine, agent framework, or simulation platform.

It does not aim to reproduce *Advance Wars* faithfully.

Configurability should not justify premature abstraction or prevent the first game from developing a strong identity of its own.

## Name Origin

Aces was originally conceived under the working title **Republic Aces**.

The name emerged from an intentionally irreverent, anti-war concept and a play on “Republic Asses”, imagining a satirical military setting with absurd and burlesque representations of warfare and the societies built around it.

As the project evolved, its scope became broader than that initial theme. The project was renamed **Aces** for simplicity and to avoid coupling its identity to a particular political, military, or visual interpretation.

The original name is preserved here as part of the project's history. **Republic Aces** may still exist in the future as a specific theme or implementation of Aces rather than as the identity of the project itself.
