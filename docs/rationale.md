# Rationale — Why OTG Exists

**Date Created:** 2026-03-23\
**Last Updated:** 2026-11-18\
**Author:** UG.\
**Covers:** OTG Syntax Specification draft 0.03

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
7. Statement of points left open in 0.02, and `first()` as a set (0.03)

Stage 6 is described below. It is the only stage in which the specification\
changed without the notation changing.

Stage 7 is described below, after stage 6.

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

## Why the Specification Was Revised in 0.03

The points below were either unstated in 0.02 or stated in a way that\
contradicted 0.02's own examples. Each lists the options that existed and the one\
adopted.

### Position of `=`

0.02's grammar attached an optional `=` to every chain, so `1-2=-3=` was\
grammatical. No meaning was stated for it, and no glyph in the device tables\
uses it.

* Define a meaning: `2 → output`, and `2` also takes part in the mix into `3`.\
  *Rejected.* `=` reads as a full stop. A structure that continues after a full\
  stop contradicts the terminating role that 0.02 made primary.
* **Allow `=` only after the last chain of a segment.** *Adopted.* `=` moves from\
  `<chain>` to the segment level in the grammar. `1-2=-3=` and `1-2-=` are\
  rejected.

### Minimum size of a parallel

0.02's grammar for a closed parallel, `<chain> "-" <chain> { "-" <chain> }`,\
admitted `1-2=`: one branch and a destination. The open parallel already required\
two branches, through its trailing `-`.

* Allow it as a one-branch mix, equivalent to the series `12=`.\
  *Rejected.* Two ways to write a series, and `-` would no longer mean that\
  something is mixed.
* **Require at least two branches.** *Adopted.* The grammar is\
  `<chain> "-" <chain> "-" <chain> { "-" <chain> }`, and sections 5.3, 5.4, 6, 7\
  and 15 state `n ≥ 3` for a closed parallel and `n ≥ 2` for an open one. No\
  table glyph has a one-branch parallel.

### `$` in nested `@(group)`

0.02 said `$` may appear anywhere inside `@(group)`, without saying which\
`@(group)` it belongs to when they are nested.

* The outermost `@(group)`.
* Every enclosing `@(group)`.
* **The nearest enclosing `@(group)`.** *Adopted.* The target is decided by the\
  group the reader is looking at, and an inner group's feedback can be read\
  without the surrounding expression.

### `@` on an operator inside `@(group)`

0.02 section 9.5 stated that `@` on an operator in `exits(group)` is read as `$`,\
and gave `@($1-2-$3-)4= ≢ @($1-2-@3-)4=`, where `3` is in `exits(group)`. The two\
cannot both hold.

* Keep the rule and correct the example to `≡`.
* **Keep the example and remove the rule.** *Adopted.* `@` on an operator is self\
  feedback everywhere, and is not counted as a `$`. This keeps *one symbol, one\
  meaning* for `@`. The equivalence `@($12$3)4= ≡ @($12@3)4=` still holds,\
  because `exits(group)` is `{3}` and both forms add `3 → 3`; it is a coincidence\
  of edges, not a reading of `@` as `$`.

With the rule removed, `@(12@3)4=` adds both `3 → 3` and the default feedback\
`3 → 1`.

### `$` on a group

The grammar allowed `$(12)`; 0.02 did not say what it means.

* Reject `$` on a group.
* **Target `first(group)`.** *Adopted.* `$(12) ≡ $12`. A feedback target is a\
  destination, and `first()` already defines where a group is entered.

### `first()` as a set

Defining `$` on a group raised the case `$(1-2-)`. Under 0.02, `first()` of a\
parallel was `first(C1)`, so `$(1-2-)` would target `1` only. The reading\
intended was `$1` and `$2`. The same question applied to a group used as a\
destination: `5(1-2-)` was not defined by intent, only by the 0.02 table.

* Distribute `$` over branches, and keep `first()` unchanged for connections.\
  *Rejected.* A feedback target and a connection destination would be decided by\
  two different definitions of where a group is entered.
* **Redefine `first()` as the set of entries.** *Adopted.* Closed parallel: the\
  entries of `C1 … Cn-1` (the destination chain `Cn` receives the mix, not\
  connections from outside). Open parallel: the entries of all chains.

This left two further choices.

For the default target of `@(group)` without `$`:

* Keep `first(C1)` for the default target only. Existing examples in sections\
  9.4 and 12 stay as written, at the cost of a second definition of entry.
* **Use the same set.** *Adopted.* One definition serves destinations, `$`\
  targets and the default feedback target. Section 9.4's `@(1-2-)3=` and section\
  12's canonical example are rewritten.

For an expr of several segments, as in `1(23=45=)`:

* `first(S1)`, as in 0.02.
* **The union over all segments.** *Adopted*, for the same reason as the\
  parallel: every segment is an entry.

The readings were fixed by eighteen worked examples (sections 6.1, 9.1, 9.4)\
before the definition was written, and the definition was checked against them.

Before adoption, two checks were made:

* **Device tables.** All 40 glyphs, as written in 0.02, give the same edge set\
  as under 0.02. None of them uses a parallel or multi-segment group as a\
  destination or feedback target.
* **Expressiveness.** Every glyph of up to eight characters over three operators\
  was parsed under both definitions. No edge set expressible under 0.02 is lost.\
  The 0.02 reading remains writable by placing the source in the first branch's\
  chain (`51-2-` for 0.02's `5(1-2-)`) or by naming the target with `$`\
  (`@($1-2-)` for 0.02's `@(1-2-)`).

This is the one change in 0.03 that gives a different structure to a glyph valid\
under both 0.02 and 0.03.

### Identifier case

* Case-insensitive.
* **Case-sensitive.** *Adopted.* `a-z` was added to `<short_operator>` in 0.01 as\
  additional identifiers, which reads `A` and `a` as distinct. The alias macro\
  layer (issue #3) adopted case-sensitive alias names on the same basis.

### Spaces

0.02 described space as padding used only for storage, without stating where it\
may appear.

* Spaces anywhere, ignored by the parser.\
  *Rejected.* `12 34` would read as two structures while parsing as one chain.
* Leading and trailing.
* **Trailing only.** *Adopted.* Space fills a fixed-length storage field after the\
  glyph and is removed before parsing. A space elsewhere is invalid. This matches\
  the alias layer, where whitespace around `:=` is prohibited.

### Glyph closure

0.02 allowed the last segment of a glyph to be open. A glyph such as `@134=23`\
was grammatical: `23` ends with no destination.

* Require every glyph to end with `=`.\
  *Rejected.* `(@1-2-3-4=)` and `@1(2=3=4=)` end with a group whose content is\
  already closed; a trailing `=` would add nothing.
* **Require every glyph to end closed: with `=`, or with a group whose own last\
  segment is closed.** *Adopted.* Inside a group the last segment may still be\
  open, as in `(1-2-)3`, since what follows the group completes it.

Examples in the specification that end open are kept as written and defined as\
fragments, since a fragment shows the part under discussion with less noise.

Under this rule, the shorter forms `@124=34` (OPM 3) and `@14=234` (OPM 2) are\
invalid. Their closed forms `@124=34=` and `@14=234=` are the same length as the\
table glyphs.

### Canonical form: occurrences of the same operator

With `first()` as a set, branching from one operator can be written with a group\
instead of repeating the operator (`@12=@13=@14=` ≡ `@1(2=3=4=)`). Section 12 did\
not say which is canonical. Its priority of fewer symbols alone selects the group\
in some table entries and the repetition in others, and it also admits\
`@124=34=` against `@12-3-4=` at equal length.

* Use grouping only where it is shorter.\
  Changes OPM 5, DX7 22 and 24. Leaves the choice between `@124=34=` and\
  `@12-3-4=` to a further rule.
* **Add a priority: fewer occurrences of the same operator, above the removal of\
  redundant symbols.** *Adopted.* One rule decides both cases: fan-out is grouped,\
  and fan-in is written with `-` rather than by repeating the destination. It\
  applies even where the grouped form is longer.

Ordering stays above it. For DX7 22, `21=@6(5=4=3=)` is shorter than\
`@6(5=4=3=)=21=`, but the device uses descending order.

Table entries rewritten: YM2151 / YM2203 algorithm 5, DX7 algorithms 19–25. Their\
structures are unchanged, and their 0.02 glyphs remain valid.

---

## Summary

OTG is not just a notation.\
It is a structural language for describing operator topology.
