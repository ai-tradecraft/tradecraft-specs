# Tradecraft Terminology

This document establishes the canonical vocabulary used throughout the Tradecraft framework. Agentic AI terminology is frequently overloaded: *agent*, *runtime*, *session*, and *orchestrator* may denote different concepts across research literature and commercial systems. Tradecraft uses deliberately distinct terms for semantic specifications, logical runtime entities, execution attempts, harness-managed state, and infrastructure responsibilities.

The definitions below are authoritative for Tradecraft documentation. Component and protocol specifications establish the corresponding behavioral and conformance requirements.

## Normative Language

The key words **MUST**, **MUST NOT**, **REQUIRED**, **SHALL**, **SHALL NOT**, **SHOULD**, **SHOULD NOT**, **RECOMMENDED**, **NOT RECOMMENDED**, **MAY**, and **OPTIONAL** in Tradecraft's normative specifications are to be interpreted as described in [BCP 14](https://www.rfc-editor.org/info/bcp14), comprising [RFC 2119](https://www.rfc-editor.org/rfc/rfc2119) and [RFC 8174](https://www.rfc-editor.org/rfc/rfc8174), when, and only when, they appear in all capitals.

These conventions apply to normative Tradecraft specifications unless a document explicitly defines a different convention. Lowercase uses of these words carry their ordinary English meanings. Explanatory examples, design rationale, and non-normative guidance do not impose requirements unless expressly designated as normative.

The use of a normative keyword does not make a requirement universally applicable: its scope is determined by the specification, component, and conditions in which it appears. Requirements should be stated in a manner that permits implementation and verification.

## Glossary

### Actor

An **Actor** is a logical runtime entity representing an instantiated agentic workload managed by Tradecraft. It has an identity and lifecycle independent of any particular execution attempt, operating-system process, or vendor-managed session.

An Actor may be instantiated from an [Agent](#agent) specification, together with a task and its resolved execution configuration. Multiple Actors can share the same Agent specification while retaining separate identities, state, and histories. An Actor may undergo multiple [Executions](#execution) as work is retried, resumed, or recovered.

The term is not a commitment to the concurrency or message-passing semantics of the classical Actor Model.

### Agent

An **Agent** is a declarative specification of the semantic role, expected behavior, capabilities, and constraints that a language model is to adopt in performing a task.

The term *agent* is overloaded in AI research and industry practice: it may refer to an autonomous program, a tool-using LLM, a worker process, or a persistent interaction context. Within Tradecraft, **Agent** specifically denotes the reusable *semantic specification*, not the running worker.

An Agent specification may include its purpose, role, behavioral instructions, communication style, skills, tool and MCP requirements, context requirements, model preferences, and operational constraints. It is independent of a particular harness, although some capabilities may require harness-specific adaptation.

The runtime entity instantiated to carry out work is an [Actor](#actor).

### Agent Harness

An **Agent Harness** is the software substrate that coordinates model interactions with tools, resources, and execution context to perform agentic work. Depending on the implementation, it may provide model invocation, tool dispatch, permission controls, context management, event hooks, session persistence, and delegation.

Examples include Claude Code, Codex, OpenCode, and Pi. Tradecraft does not presume that all harnesses have equivalent APIs, state models, or lifecycle capabilities. Harness-specific operations are mediated through a [Lamp](#lamp).

### Execution

An **Execution** is a discrete, identifiable attempt to perform work on behalf of an [Actor](#actor). It captures the operational history and outcome of that attempt, including its resolved configuration, state transitions, environment association, observations, and results.

An Actor may have multiple Executions, for example following a retry or recovery. An Execution is not necessarily equivalent to one process, one model call, or one harness [Session](#session). A single Execution may interact with multiple Sessions when the selected harness and orchestration strategy permit it.

Execution outcomes may include success, failure, cancellation, or other outcomes defined by the orchestration specification. The precise state machine, retry semantics, and durability guarantees are defined elsewhere.

### Lamp

**L**iaison for **A**ctors, **M**etadata, and **P**latforms.

A **Lamp** is a harness-specific SDK or integration adapter that provides Tradecraft with a consistent control and observation interface for a particular [Agent Harness](#agent-harness).

A Lamp translates framework-level operations into harness-specific mechanisms, such as command-line invocations, APIs, configuration files, or event hooks. Its responsibilities may include:

- Starting, attaching to, interrupting, resuming, and terminating harness activity, where supported.
- Translating execution specifications and harness configuration.
- Mapping Tradecraft identities to native harness Session identifiers.
- Normalizing events, status, outputs, and execution metadata.
- Exposing harness capabilities, limitations, and extension points.
- Relaying permission requests, clarification requests, and other human-interaction events.

A Lamp is an integration boundary, not the authority for defining Agent semantics or deciding the overall orchestration workflow. It may assist environment preparation with harness-specific knowledge, but environment provisioning is the responsibility of the [Lamplighter](#lamplighter).

### Lamplighter

A **Lamplighter** is the component responsible for preparing, configuring, and validating the environment required to execute an Actor's workload.

It transforms an execution specification into an operationally suitable environment. Depending on the execution substrate, its responsibilities may include workspace creation, source checkout, dependency preparation, context materialization, Agent and skill configuration, MCP integration setup, credential and permission provisioning, and prerequisite validation.

A Lamplighter may consult a [Lamp](#lamp) to determine harness-specific requirements or materialize harness-specific configuration. The responsibilities remain distinct: **the Lamplighter prepares the conditions for execution; the Lamp mediates control of the harness.**

### Session

A **Session** is a stateful interaction context established or maintained by an [Agent Harness](#agent-harness). It may preserve conversational history, tool interactions, intermediate artifacts, and other context needed to continue work.

Session identity, persistence, resumption, and termination semantics are harness-specific. A Session is distinct from both an [Actor](#actor) and an [Execution](#execution). An Execution may involve multiple Sessions, and a Session may be reused across Executions if permitted by the harness and Tradecraft's execution policies.

A Session is not, by itself, the authoritative record of an Actor's lifecycle or Tradecraft's durable orchestration state. The [Lamp](#lamp) mediates mappings between Tradecraft identities and harness-native Session identities.

## Conceptual Relationships

| Concept | Architectural nature | Primary responsibility |
| --- | --- | --- |
| **Agent** | Declarative specification | Defines semantic role, behavior, capabilities, and constraints. |
| **Actor** | Logical runtime entity | Represents an instantiated workload with stable identity and managed lifecycle. |
| **Execution** | Execution attempt | Records an identifiable attempt to perform work on behalf of an Actor. |
| **Session** | Harness-managed interaction context | Preserves conversational or operational continuity within a harness. |
| **Agent Harness** | Execution substrate | Coordinates model, tools, and environment interactions. |
| **Lamp** | Vendor/harness integration SDK | Translates Tradecraft control and observation operations to harness-specific mechanisms. |
| **Lamplighter** | Environment provisioner | Prepares and validates the resources and configuration required for execution. |

A typical relationship is:

```text
Agent specification
        |
        | instantiated for a task
        v
      Actor
        |
        | one or more attempts
        v
    Execution(s)  <------ Tradecraft orchestration
        |
        | prepared environment
        +<---------------- Lamplighter
        |
        | harness control via
        v
       Lamp  <----------> Agent Harness
                                  |
                                  | maintains
                                  v
                              Session(s)
```

This diagram is conceptual, not a prescription for process boundaries or cardinalities.

## Architectural Principles

The following principles establish the conceptual boundaries of the framework. Specific, testable obligations belong in the relevant normative component and protocol specifications.

1. **Specification and runtime identity are separate.** An Agent describes a semantic role; an Actor represents a logical instantiated workload.
2. **Logical identity and execution attempts are separate.** An Actor can persist across multiple Executions without losing its identity.
3. **Tradecraft identity and harness identity are separate.** Actor and Execution identifiers are not interchangeable with native Session identifiers.
4. **Runtime state does not implicitly rewrite semantic specifications.** Information accumulated by an Actor or Session does not, by itself, modify its source Agent specification.
5. **Harness integration is explicit.** Lamps isolate vendor-specific mechanisms and expose differences in supported capabilities rather than pretending that all harnesses behave identically.
6. **Provisioning and control are separate concerns.** Lamplighters prepare environments; Lamps control and observe harness interactions.
7. **Durability belongs to Tradecraft's orchestration model.** Harness session persistence can support recovery but does not replace framework-managed execution history and state.

## References

- [RFC 2119 — Key words for use in RFCs to Indicate Requirement Levels](https://www.rfc-editor.org/rfc/rfc2119)
- [RFC 8174 — Ambiguity of Uppercase vs Lowercase in RFC 2119 Key Words](https://www.rfc-editor.org/rfc/rfc8174)
- [BCP 14 — Requirement Levels](https://www.rfc-editor.org/info/bcp14)
