# Rationale — Why OTG Exists

**Date Created:** 2026-03-23\
**Last Updated:** 2026-08-03\
**Author:** UG.\
**Covers:** OTG Syntax Specification draft 0.02

---

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

OTG was originally developed during the design of **OPM Tone Editor (a custom FM\
tone editor developed by the author)** and future OED (FM tone library format).

The goal was to:

* Make operator topology explicit
* Enable machine interpretation (AST / graph)
* Provide a compact and readable notation
* Support future extensibility

---

## Origin

OTG did not begin as a language design. It began as a wish:

→ "I want to see an FM algorithm at a glance."

The sequence was:

1. **Wish.** Show the topology directly, not as an index into a table.
2. **Constraint.** Roughly ten characters. ASCII only. Readable without a legend.
3. **Sketch.** Written in a text editor, by hand.\
   Series was obvious — `1234=`.
4. **Break it.** "What about parallel?" The first sketch could not express it.\
   `-` was introduced and tried: `1-2-34=`.
5. **Enumerate.** All eight YM2151 algorithms were written out.\
   Not a sample — the complete set.
6. **Formalize.** Only after all eight held together was the notation treated as\
   a candidate for a specification.

Step 5 is the step that mattered. A notation that works for one example proves\
nothing; a reader can always find a second reading. Eight is not a large number,\
but for this device it is *all* of them, and within a closed set the entries\
constrain each other. Once `-` exists in some glyphs, a reading that ignores `-`\
in the others becomes untenable. The set makes the reading unique.

This is also why the DX7 table was written next. Thirty-two algorithms over six\
operators is a different problem from eight over four, and it is what forced\
grouping `( )` and the feedback target `$` to exist.

---

## Design Principles

OTG was designed with the following principles:

* ASCII-only representation
* Minimal symbol set
* Structure-first (not parameter-based)
* One symbol, one meaning
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
6. Correction of the written grammar to match the notation in use (0.02)

Stage 6 is described below. It is the only stage in which the specification\
changed without the notation changing.

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

## Visual Design Philosophy

OTG was not designed as a purely formal language.

Instead, it was designed with the following idea:

→ "The structure should be guessable by just looking at it."

In OPM Tone Editor, OTG expressions were used without any explanation.\
Users familiar with FM synthesis could intuitively understand the topology from\
the glyph itself.

### Visual Meaning of Symbols

Each symbol was chosen with visual intuition in mind:

* `-` : parallel (side-by-side)
* `=` : output / termination
* `@` : feedback (loop-like association)
* `( )` : grouping (scope)
* `$` : target indication

The goal is not only correctness, but also **immediate visual interpretation**.

### Design Balance

OTG intentionally balances:

* Human intuition
* Structural correctness
* Machine interpretability

It is not purely formal, and not purely informal.

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

Canonical form in OTG is not only for normalization.\
It is also intended to improve visual readability.

OTG expressions should be:

* compact
* consistent
* easy to scan visually

This reflects the original design goal:

→ "The structure should be understandable at a glance."

---

## Why the Grammar Was Corrected in 0.02

### What was wrong

Draft 0.01 described a language that was not the language in use. Three of its\
rules contradicted the algorithm tables published on the same day:

* **Binding.** 0.01 made `-` bind tighter than concatenation, and restricted the\
  operands of `-` to single operators or groups. Under that rule `@1-23-4=`\
  (OPM alg 2) reads as a plain series, which is not alg 2.
* **Termination.** 0.01 treated `=` purely as a carrier marker on an operator.\
  Under that rule `@12=34=` (OPM alg 4) reads as one chain of four operators.
* **Identity and repetition.** 0.01 said nothing about what happens when the same\
  identifier appears more than once. OPM alg 5 (`@12=@13=@14=`) depends on it.

The tables were correct. The grammar was not.

### Why it happened

The notation was derived visually and then validated against a closed set of\
instances. The grammar was written afterward, as a *description* of that\
notation, and was never put through the same validation.

Enumerating all eight algorithms tested the glyphs. Nothing tested the BNF.

That is the whole of it. The method that produced a reliable notation was simply\
not applied a second time to the formalization of it.

### Why the visual reading was right and the BNF was wrong

Worth recording, because it is the reverse of what one might expect from a\
"formalize the intuition" exercise.

Adjacency with no separator binds more tightly than an explicit separator — this\
is how `ab + cd` is read, and it is how `65-43-2-1=` is read. The visual design\
had this correct from the first sketch. The error entered when the reading was\
transcribed into BNF, where `-` was treated as an infix operator over atoms\
because that is the shape infix operators usually have.

The formal layer drifted from the intuitive layer. The intuitive layer was the\
accurate one.

### Alternatives considered

**For binding:**

* Require explicit grouping for multi-operator branches — `1-(23)-4=`.\
  *Rejected.* Every existing glyph would have to change, and the added\
  parentheses work directly against reading at a glance.
* Introduce a second separator distinguishing "branch" from "chain".\
  *Rejected.* Symbol set growth, for a distinction the eye already makes.
* **Widen the operands of `-` from atoms to chains.** *Adopted.* No glyph changes;\
  the rule now says what readers were already doing.

**For termination:**

* Introduce a segment separator symbol.\
  *Rejected.* A new symbol for something `=` already communicates. `=` reads as a\
  full stop.
* Require each segment to be a group — `(12=)(34=)`.\
  *Rejected.* Verbose; the parentheses carry no information.
* **Define `=` as a chain terminator, with output routing as its consequence.**\
  *Adopted.*

The last point deserves care, because *one symbol, one meaning* is a stated\
design principle and `=` appears to carry two. The resolution is that termination\
is the single meaning: `=` closes a chain, and a closed chain's exits are by\
definition what the device emits. Output is not a second role but the consequence\
of the first. Had the two been genuinely independent, a second symbol would have\
been the correct answer.

**For identity:**

* Require each identifier to appear once, with explicit fan-out notation.\
  *Rejected.* OPM alg 5 and eight of the DX7 algorithms would require grouping\
  that expresses nothing the reader does not already infer.
* **State that an identifier denotes a node, not an occurrence.** *Adopted.*\
  This in turn requires edges to form a set, so that a feedback declared in\
  several places is applied once.

### Why the reference tables are informative, not normative

They were used as ground truth during this correction, and they are the material\
against which any parser should be checked. That argues for making them\
normative.

They are not, for one reason: OTG is device-independent, and the tables are not.\
Binding the language to YM2151 and DX7 would make every future device a\
specification amendment. The tables verify the specification; they do not\
constitute it.

Their practical role is regression material. That role does not require normative\
status.

### What this leaves

The notation is unchanged. Every glyph written since March 2026 remains valid and\
means what it meant.

What changed is that the written grammar now describes it. An implementer working\
from draft 0.02 alone, with no access to the tables and no prior familiarity with\
FM topology, will produce the same readings as a reader working from the glyphs\
by eye.

That equivalence — formal reading and visual reading agreeing — was the goal from\
the first sketch. Until 0.02 it held in the notation but not in the document.

---

## Summary

OTG is not just a notation.\
It is a structural language for describing operator topology.
