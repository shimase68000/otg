# Rationale — Why OTG Exists

## Background

Traditional FM synthesizer algorithms are typically represented by numeric IDs.

However:

* The structure is not visible
* The topology must be inferred from documentation
* The representation is not extensible

---

## Problem

Numeric algorithm representations have several limitations:

* Opaque (no structural meaning)
* Device-specific (not portable)
* Hard to extend beyond predefined algorithms

---

## Motivation

OTG was originally developed during the design of **OPM Tone Editor (a custom FM tone editor developed by the author)** and future OED (FM tone library format).

The goal was to:

* Make operator topology explicit
* Enable machine interpretation (AST / graph)
* Provide a compact and readable notation
* Support future extensibility

---

## Design Principles

OTG was designed with the following principles:

* ASCII-only representation
* Minimal symbol set
* Structure-first (not parameter-based)
* Human-readable, but not beginner-oriented
* Machine-friendly (parser / graph conversion)
* Device-independent

---

## Key Insight

Instead of:

→ "Algorithm = number"

OTG defines:

→ "Algorithm = structure"

---

## Evolution

OTG evolved in the following stages:

1. Internal notation for OPM Tone Editor
2. Consideration for OED001 integration
3. Formalization as a DSL
4. Introduction of feedback target (`$`)
5. Definition of canonical form

---

## Scope

OTG is primarily designed for FM operator topology.

However, it is intentionally defined in a way that may support:

* different operator counts
* non-FM topology systems
* future extensions

---

## Relationship to OED

OTG was initially developed in the context of OED.

However:

* OTG is defined as a standalone DSL
* It does not depend on OED
* It can be used independently

---

## Summary

OTG is not just a notation.

It is a structural language for describing operator topology.

---

## Visual Design Philosophy

OTG was not designed as a purely formal language.

Instead, it was designed with the following idea:

→ "The structure should be guessable by just looking at it."

In OPM Tone Editor, OTG expressions were used without any explanation.

Users familiar with FM synthesis could intuitively understand the topology from the glyph itself.

---

### Visual Meaning of Symbols

Each symbol was chosen with visual intuition in mind:

* `-` : parallel (side-by-side)
* `=` : output / termination
* `@` : feedback (loop-like association)
* `( )` : grouping (scope)
* `$` : target indication

The goal is not only correctness, but also **immediate visual interpretation**.

---

### Design Balance

OTG intentionally balances:

* Human intuition
* Structural correctness
* Machine interpretability

It is not purely formal, and not purely informal.

---

### Guiding Principle

OTG is designed so that:

→ A human can roughly understand the topology at a glance
→ A machine can interpret it precisely

---

## Implication

This design approach makes OTG:

* easy to read for experienced users
* expressive without verbosity
* suitable for both UI display and data representation

---

## Canonical Form Philosophy

Canonical form in OTG is not only for normalization.

It is also intended to improve visual readability.

OTG expressions should be:

- compact
- consistent
- easy to scan visually

This reflects the original design goal:

→ "The structure should be understandable at a glance."
