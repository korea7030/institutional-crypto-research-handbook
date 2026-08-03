ARCHITECTURE.md

The Institutional Crypto Research Handbook

Knowledge Architecture Specification

Version: v1.0

Status: Foundation

⸻

Purpose

This document defines the architecture of the knowledge system used throughout The Institutional Crypto Research Handbook.

Unlike traditional books, this project is designed as a Living Knowledge Base.

Every chapter, concept, case study, and research document is part of a single interconnected knowledge graph.

This architecture is designed for both:

* Human learning
* AI-assisted knowledge retrieval

⸻

Design Principles

The architecture follows seven foundational principles.

Principle 1 — Knowledge Before Documents

Documents are containers.

Knowledge is the primary asset.

A document may change.

Knowledge should remain stable.

⸻

Principle 2 — Single Source of Truth

Every concept must have exactly one authoritative definition.

Example:

Gas is defined only once.

Every other document references that definition.

Duplicate definitions are prohibited.

⸻

Principle 3 — Atomic Knowledge

Large documents should be decomposed into reusable knowledge objects.

Instead of

Ethereum.md

Use

* Account
* Transaction
* Gas
* Storage
* Logs
* Receipt
* State Transition

Each object should remain independently understandable.

⸻

Principle 4 — Stable Identity

Every knowledge object receives a permanent identifier.

Examples

BTC-UTXO-001

ETH-TX-001

ETH-GAS-001

ETH-STATE-001

DEFI-AMM-001

Identifiers never change.

Document titles may evolve.

Knowledge IDs do not.

⸻

Principle 5 — Relationships First

Knowledge gains value through relationships.

Every object should define:

Requires

Related To

Used By

Extends

Contrasts With

Replaced By

Understanding relationships is more important than memorizing isolated facts.

⸻

Principle 6 — AI-First Structure

The repository should be readable by both humans and machines.

Documents must support:

* Semantic search
* RAG systems
* Knowledge graphs
* AI agents
* Static documentation sites

Human readability must never be sacrificed.

⸻

Principle 7 — Continuous Evolution

Knowledge is never complete.

Every object has a lifecycle.

Draft

↓

Reviewed

↓

Stable

↓

Deprecated

↓

Archived

The repository represents living knowledge.

⸻

Knowledge Object Model (KOM)

Every concept is represented as a Knowledge Object.

A Knowledge Object contains:

* Identifier
* Canonical Name
* Definition
* Relationships
* Metadata
* References
* Status
* Version

Knowledge Objects are the smallest reusable unit in the repository.

⸻

Knowledge Object Metadata

Every object should contain:

knowledge_id:
canonical_name:
aliases:
category:
difficulty:
status:
version:
owner:
last_reviewed:
prerequisites:
related_objects:
primary_sources:
secondary_sources:
tags:

Metadata is mandatory.

⸻

Repository Layers

The repository is organized into layers.

Layer 1

Foundation

Layer 2

Protocols

Layer 3

Applications

Layer 4

Metrics

Layer 5

Research

Layer 6

Case Studies

Layer 7

Professional Practice

Knowledge should always flow upward.

⸻

Dependency Rules

Dependencies must be explicit.

Example

ETH-GAS-001

requires

ETH-TX-001

ETH-TX-001

requires

ETH-ACCOUNT-001

Circular dependencies should be avoided.

⸻

Knowledge Graph

Every object belongs to a graph.

Relationships include

Requires

Depends On

Uses

Produces

Related To

Contrasts With

Successor Of

Predecessor Of

This graph forms the conceptual backbone of the project.

⸻

Naming Convention

Knowledge IDs follow a predictable format.

DOMAIN-TOPIC-NUMBER

Examples

BTC-UTXO-001

ETH-GAS-001

ETH-EVM-001

DEFI-AMM-001

METRIC-MVRV-001

CASE-FTX-001

Identifiers remain immutable.

⸻

Folder Architecture

chapters/
concepts/
case-studies/
frameworks/
playbooks/
references/
assets/
templates/
glossary/
meta/

Concepts should remain independent from presentation.

⸻

Knowledge Lifecycle

Every object progresses through defined stages.

Draft

Initial authoring.

Reviewed

Peer-reviewed.

Stable

Approved for reference.

Deprecated

Concept superseded.

Archived

Historical record retained.

Deprecated objects should always reference their replacement.

⸻

Traceability

Every important conclusion must support backward traceability.

Conclusion

↓

Evidence

↓

Dataset

↓

Primary Source

↓

Protocol Specification

Readers should always be able to reconstruct the reasoning process.

⸻

Versioning

Knowledge Objects

MAJOR.MINOR.PATCH

Repository

Semantic Versioning

Major conceptual changes require version increments.

⸻

Human Readability

Documents should prioritize:

* Clear hierarchy
* Consistent terminology
* Minimal ambiguity
* Explicit assumptions
* Stable navigation

⸻

AI Readability

Documents should support:

* Structured headings
* Stable identifiers
* Metadata-first organization
* Explicit relationships
* Consistent terminology
* Atomic concepts

The architecture intentionally avoids hidden context.

⸻

Future Compatibility

The architecture is designed to support:

* GitHub
* GitHub Pages
* Docusaurus
* MkDocs
* Obsidian
* Semantic Search
* Vector Databases
* Retrieval-Augmented Generation (RAG)
* Autonomous AI Research Agents

No platform-specific assumptions should be embedded into the knowledge itself.

⸻

Architectural Decision Records (ADR)

Major architectural decisions should be documented separately.

Each ADR should include:

* Context
* Decision
* Alternatives Considered
* Consequences
* Status

Architectural changes must be transparent.

⸻

Success Criteria

The architecture succeeds when:

* Every concept has a single authoritative definition.
* Every conclusion is traceable.
* Every relationship is explicit.
* Knowledge remains reusable.
* AI systems can navigate the repository.
* Humans can learn progressively.
* New contributors can extend the project without breaking consistency.

⸻

Guiding Principle

A great handbook teaches.

A great architecture continues teaching long after it is written.

The Institutional Crypto Research Handbook is designed not as a collection of documents, but as a continuously evolving institutional knowledge system.
