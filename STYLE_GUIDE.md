STYLE_GUIDE.md

The Institutional Crypto Research Handbook

Documentation & Writing Standard

Version: v1.0 (Foundation Standard)

⸻

Purpose

This document defines the writing, documentation, and research standards used throughout The Institutional Crypto Research Handbook.

Its purpose is to ensure that every chapter maintains the same level of rigor, structure, readability, and evidence quality.

This is not merely a writing guide.

It is an institutional documentation standard.

⸻

Core Philosophy

This handbook is built upon five principles.

1. Facts Before Interpretation

Facts must always appear before any interpretation.

Readers should be able to distinguish between:

* What is objectively observable.
* What is inferred.
* What remains uncertain.

⸻

2. Explain Why Before How

Every major concept should answer:

* Why does this exist?
* Which problem does it solve?
* What assumptions does it make?
* What limitations does it have?

Implementation details should never appear before motivation.

⸻

3. Evidence Over Confidence

Confidence must never exceed available evidence.

The strength of every important claim should match the quality of supporting evidence.

⸻

4. Historical Context Matters

Every technology exists because of a historical problem.

Explain:

* Previous limitations
* Design motivation
* Evolution
* Current trade-offs

⸻

5. Institutional Thinking

Every chapter must include the question:

How would an institutional researcher evaluate this evidence?

⸻

Chapter Structure Standard

Every chapter follows the same structure.

1. Metadata
2. Learning Objectives
3. Prerequisites
4. Key Questions
5. Definitions
6. Historical Context
7. Technical Foundations
8. Institutional Thinking
9. Case Studies
10. Common Misconceptions
11. Research Questions
12. Challenge Section
13. Summary
14. Further Reading
15. References

This structure should remain consistent across all chapters.

⸻

Metadata Standard

Every document begins with YAML metadata.

Example:

---
title:
chapter:
version:
status:
difficulty:
estimated_reading:
estimated_study:
last_reviewed:
prerequisites:
related_topics:
primary_sources:
secondary_sources:
tags:
---

Metadata should describe the document, not repeat its contents.

⸻

Writing Principles

One Paragraph = One Idea

Each paragraph should communicate only one primary idea.

⸻

Define Before Discussing

Always define terminology before using it.

Undefined concepts create ambiguity.

⸻

Prefer Precision Over Simplicity

Simplification must never sacrifice correctness.

⸻

Avoid Absolute Statements

Avoid words such as:

* Always
* Never
* Guaranteed

Unless directly supported by a formal specification.

⸻

State Assumptions Explicitly

Every model, metric, and conclusion relies on assumptions.

Those assumptions must be clearly identified.

⸻

Evidence Classification

Every major statement should be categorized.

FACT

Objectively verifiable information.

Supported by:

* Official documentation
* Protocol specifications
* Standards
* Whitepapers

⸻

INTERPRETATION

A reasoned explanation derived from evidence.

Interpretations should never be presented as facts.

⸻

HYPOTHESIS

A proposed explanation requiring further validation.

Hypotheses must identify:

* Supporting evidence
* Missing evidence
* Potential falsification

⸻

CONTROVERSY

Used when significant disagreement exists among researchers.

Conflicting viewpoints should be documented fairly.

⸻

UNKNOWN

Used when evidence is insufficient.

No speculation should replace missing evidence.

⸻

Evidence Confidence Level (ECL)

Every significant claim should include an Evidence Confidence Level.

ECL-A — Primary Evidence

Supported directly by:

* Official specifications
* BIPs
* EIPs
* Whitepapers
* Protocol documentation

Highest confidence.

⸻

ECL-B — Strong Independent Evidence

Supported by multiple independent sources.

Examples:

* Peer-reviewed papers
* Independent protocol analyses
* Multiple consistent primary datasets

⸻

ECL-C — Accepted Industry Interpretation

Commonly accepted by professional researchers.

May not be formally specified.

Should be identified as interpretation.

⸻

ECL-D — Emerging Hypothesis

Supported by limited evidence.

Useful for research.

Not suitable as an established conclusion.

⸻

ECL-U — Unknown

Evidence currently insufficient.

Further research required.

⸻

Source Hierarchy

Priority order:

Tier 1

Official Documentation

Tier 2

Protocol Specifications

Tier 3

Academic Papers

Tier 4

Protocol Source Code

Tier 5

Institutional Research

Tier 6

Community Research

Lower-tier sources should never override higher-tier evidence without explicit justification.

⸻

Citation Standard

Every important claim should be traceable.

Preferred citation order:

1. Official documentation
2. Standards (BIP / EIP)
3. Academic papers
4. Protocol repositories
5. Institutional research
6. Community research

Avoid unsupported assertions.

⸻

Diagram Standard

Diagrams should explain concepts rather than decorate documents.

Principles:

* Minimalist
* Consistent colors
* Directional flow
* Clear labels
* Accessible without surrounding text

Diagram types:

* Architecture
* Data Flow
* State Transition
* Timeline
* Research Framework
* Decision Tree

⸻

Institutional Thinking Box

Every chapter must contain at least one Institutional Thinking section.

Purpose:

Explain how professional researchers would approach the topic.

⸻

Research Trap Box

Every chapter must contain at least one Research Trap.

Purpose:

Highlight common analytical mistakes.

⸻

Counter Evidence Box

Every analytical conclusion should ask:

What evidence would invalidate this conclusion?

⸻

Open Question Box

Some problems remain unsolved.

These should be explicitly documented.

⸻

Challenge Section

Every chapter ends with questions encouraging readers to challenge the presented ideas.

Good researchers question assumptions.

Great researchers question their own assumptions.

⸻

Difficulty Levels

L100

Foundational

L200

Intermediate

L300

Advanced

L400

Research

L500

Institutional

Difficulty reflects required background knowledge rather than complexity alone.

⸻

Versioning

0.x

Draft

1.x

Stable

2.x

Major Revision

Every major revision should document significant conceptual changes.

⸻

Language Style

Preferred:

* Clear
* Precise
* Evidence-based
* Neutral
* Professional

Avoid:

* Promotional language
* Emotional language
* Certainty without evidence
* Unsupported predictions

⸻

Documentation Quality Gates

A chapter is considered complete only if it satisfies all of the following:

* Metadata completed
* Learning objectives defined
* Prerequisites identified
* Definitions verified
* Primary sources cited
* Evidence levels assigned
* Institutional Thinking included
* Research Trap included
* Counter Evidence discussed
* Challenge questions provided
* References reviewed

⸻

Guiding Principle

This handbook is not designed to help readers memorize blockchain concepts.

It is designed to help readers develop the thinking process used by professional researchers.

Every chapter should leave the reader with better questions rather than greater certainty.
