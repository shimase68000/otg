# OED Operator Topology Glyph (connection_glyph)

**Syntax Specification — draft version 0.03**

**Last Updated:** 2026-11-18\
**Author:** UG.

> **Note on this revision**
> Version 0.03 fixes points that draft 0.02 left unstated: where `=` may appear\
> (4.2), which `@(group)` a `$` belongs to when groups are nested and what `$` on a\
> group means (9.4), case sensitivity of identifiers (3.2), and where spaces may\
> appear (2). It also removes a rule in 9.5 that contradicted the example given\
> with it, requires a glyph to end closed (4.5), and adds a canonical-form priority\
> for fewer occurrences of the same operator (12).
>
> One change alters meaning: `first()` becomes a set (Section 6). A parallel or a\
> multi-segment group used as a destination, as a `$` target, or under `@( )` now\
> connects to every branch entry instead of the first branch only.
>
> Every algorithm in the YM2151 / YM2203 and DX7 tables keeps its structure. Eight\
> table entries are rewritten in canonical form under section 12 (OPM 5, DX7\
> 19–25); the 0.02 glyphs remain valid and equivalent. See Section 21 for the change log and Section 22 for the\
> conformance vectors.

---

## 1. Overview

`connection_glyph` is a notation for representing operator connection structures\
(algorithms) of FM synthesis using compact ASCII strings.

This specification can express the following:

* series connections
* parallel mixing
* open parallel (incomplete parallel)
* final output (carrier)
* self feedback
* group feedback
* feedback target specification
* nested structures

---

## 2. Character Set

```
0-9, A-Z, a-z   operator identifier
@               feedback
=               segment terminator / output
-               parallel connection
( )             grouping
[ ]             multi-digit / named operator identifier
$               feedback target
space           padding (used only for storage)
```

### 2.1 Spaces

A space may appear only as trailing padding after the last character of a glyph,\
to fill a fixed-length storage field (`glyph_length`). Trailing spaces are removed\
before parsing. A space at any other position makes the glyph invalid.

```
@1234=␣␣␣␣   valid    (trailing padding; parsed as @1234=)
@12= 34=        invalid  (space inside the glyph)
```

---

## 3. Lexical Elements

### 3.1 Operator

```
<operator>       ::= <short_operator> | <long_operator>
<short_operator> ::= [0-9A-Za-z]
<long_operator>  ::= "[" <identifier> "]"
<identifier>     ::= <id_char> { <id_char> }
<id_char>        ::= [0-9A-Za-z_]
```

### 3.2 Operator Identity

An operator identifier denotes a node, not an occurrence.

Every appearance of the same identifier within a single glyph refers to the same operator.\
This makes fan-out expressible without grouping:

```
@12=@13=@14=
```

```
1 → 2 → output
1 → 3 → output
1 → 4 → output
1 → 1          (feedback, declared once, applied once)
```

`[10]` and `10` are distinct identifiers. `[1]` and `1` denote the same operator.\
In general, a `<long_operator>` whose identifier is a single character denotes the\
same operator as the `<short_operator>` of that character.

Identifiers are case-sensitive. `A` and `a` are distinct operators, and `[alpha]`\
and `[Alpha]` are distinct operators.

### 3.3 Symbols

```
@    feedback
=    segment terminator / output
-    parallel
( )  grouping
[ ]  multi-digit / named operator identifier
$    feedback target
```

---

## 4. Grammar (Informal BNF)

```
<glyph>         ::= <expr>

<expr>          ::= { <segment> "=" } [ <segment> | <open_parallel> ]

<segment>       ::= <parallel> | <chain>

<parallel>      ::= <chain> "-" <chain> "-" <chain> { "-" <chain> }   ; closed parallel
<open_parallel> ::= <chain> "-" <chain> { "-" <chain> } "-"        ; open parallel

<chain>         ::= <atom> { <atom> }

<atom>          ::= [ <prefix> ] ( <operator> | <group> )
<prefix>        ::= "@" | "$"

<group>         ::= "(" <expr> ")"
```

### 4.1 Structural Levels

The notation has four structural levels. From tightest to loosest binding:

| Level | Element              | Formed by            |
| ----- | -------------------- | -------------------- |
| 1     | atom                 | operator or `( … )`  |
| 2     | chain (series)       | concatenation        |
| 3     | parallel             | `-`                  |
| 4     | segment sequence     | juxtaposition        |

**Concatenation binds tighter than `-`.**

```
1-23-4=
```

parses as a parallel of three chains — `1`, `23`, `4` — closed by `=`, not as a\
parallel of the atoms `1` and `2` followed by `3` and `4`.

`( … )` binds tighter than everything; `@` and `$` bind to the single atom that follows them.

### 4.2 Chain Termination

A `<chain>` ends at the first of:

* `=`   (the chain is the last chain of its segment; `=` closes the segment)
* `-`   (the chain is one operand of a parallel)
* `)`   (end of enclosing group)
* end of glyph

`=` closes the segment that contains the chain. For a single chain the segment is\
the chain itself; for a closed parallel it is the whole parallel, and `=` follows\
its destination chain.

`=` therefore serves two purposes at once: it closes the segment, and it routes the\
segment's exits to the device output. The closing role is primary — it is what\
makes the following expression a new structure rather than a continuation.

`=` may appear only after the last chain of a segment. A `-` cannot follow `=`,\
since the segment is already closed:

```
@1-23-4=     valid    (= follows the destination chain 4)
1-2-3=-4=    invalid  (- after =: the segment is already closed)
1-2-=        invalid  (= after an open parallel)
```

`1-2=-3=` is invalid under either of two readings. Read as a parallel of `1`, `2`\
and `3`, it has `=` inside the parallel. Read from the left, `1-2=` is already a\
closed parallel with one branch (5.3). Both readings place the violation at the\
same `=`. A parser reading left to right reaches that `=` after two chains and\
rejects it there, so it reports the second reading.

```
@12=34=
```

```
1 → 1
1 → 2 → output
3 → 4 → output
```

Without the terminating role of `=`, this would read as a single chain\
`1 → 2 → 3 → 4`.

### 4.3 Segment Sequencing

Within an `<expr>`, every segment except the last must be closed by `=`. A segment\
ends only at `=`; without `=`, the atoms that follow continue the current chain\
instead of beginning a new segment.

```
@12=3=4=     three closed segments
@1234=       one chain; 12 followed by 34 without = is a continuation
```

### 4.4 Prefix Binding Rule

`@` and `$` apply only to the immediately following atom.

```
@12  ≡  (@1)2
```

`@` and `$` are mutually exclusive on the same atom.

### 4.5 Glyph Closure

A glyph must end closed. Its last segment is *closed* if either:

* it ends with `=`, or
* it ends with a group whose own last segment is closed (applied recursively).

A glyph that ends with an open chain has an end with no destination, and is\
invalid. Inside a group, the last segment may be open; the group is completed by\
what follows it.

```
@1234=        valid    (ends with =)
@1(2=3=4=)    valid    (ends with a group whose last segment is closed)
(@1-2-3-4=)   valid    (same)
@134=23       invalid  (23 has no destination)
@124=34       invalid  (34 is not closed)
```

**Fragments.** Examples in this specification that do not end closed — such as\
`@12`, `@A`, `(A-B-C-)D`, `@(1-$2-)`, and those labelled *fragments* — show a\
part of a glyph. They are not complete glyphs.

---

## 5. Core Semantics

Semantics are defined as a set of directed edges over operator nodes, plus a set\
of edges to the device output. Because edges form a *set*, declaring the same\
edge more than once has no additional effect (see 5.6).

`first()` and `exits()` are sets of nodes (Sections 6, 7). Throughout this\
specification, `X → first(Y)` means an edge from `X` to each element of\
`first(Y)`, and `mix(…) → first(Y)` means the mix is connected to each element of\
`first(Y)`.

### 5.1 Series

```
A B
```

```
exits(A) → first(B)
```

Example:

```
1234=
1 → 2 → 3 → 4 → output
```

### 5.2 Carrier / Termination

```
A=
```

```
exits(A) → output
```

and the segment `A` is closed. `A` is a chain or a closed parallel.

### 5.3 Parallel (Closed)

```
C1 - C2 - … - Cn
```

```
mix( exits(C1), exits(C2), …, exits(Cn-1) ) → first(Cn)
```

where each `Ci` is a chain and `n ≥ 3`. A closed parallel has at least two\
branches `C1 … Cn-1` and one destination chain `Cn`. A single branch followed by a\
destination, as in `1-2=`, is not a parallel and is rejected; the series `12=`\
expresses it. When every chain is a single operator this reduces to\
the familiar form:

```
A-B-C-D   ⇒   mix(A, B, C) → D
```

With multi-operator chains:

```
1-23-4=   ⇒   2 → 3,  mix(1, 3) → 4 → output
```

### 5.4 Parallel (Open)

```
C1 - C2 - … - Cn -
```

```
mix( exits(C1), …, exits(Cn) )
```

where each `Ci` is a chain and `n ≥ 2`, with no destination. An open parallel is incomplete: it must be enclosed in a group\
that is followed by a further connection, or it must be the operand of a group feedback.

### 5.5 Self Feedback

```
@A
```

```
A → A
```

### 5.6 Edge Set Semantics

Connections and feedback edges form a set. A duplicate declaration of an existing\
edge is a no-op.

```
@12=@13=      1 → 1  is declared twice, applied once
```

### 5.7 Group

```
(expr)
```

A group forms a syntactic scope and serves as the evaluation unit for `first()` and `exits()`.

### 5.8 Group Feedback

```
@(group)
```

```
mix( exits(group) ) → first(group)
```

---

## 6. Definition of `first()`

`first(X)` is the set of nodes that receive incoming connections when `X` is used\
as a destination. The same set is the feedback target of `$X` (9.4) and the\
default feedback target of `@X` when `X` is a group (5.8, 9.1).

**Recursive definition:**

| X                          | first(X)                                  |
| -------------------------- | ----------------------------------------- |
| operator                   | { that operator }                         |
| group `( expr )`           | `first(expr)`                             |
| chain `A B C`              | `first(A)`                                |
| closed parallel `C1-…-Cn`  | `first(C1) ∪ … ∪ first(Cn-1)`  (n ≥ 3)    |
| open parallel `C1-…-Cn-`   | `first(C1) ∪ … ∪ first(Cn)`  (n ≥ 2)      |
| expr `S1 S2 … Sn`          | `first(S1) ∪ … ∪ first(Sn)`               |

In an expr, `S1 … Sn-1` are closed segments, and `Sn` is a segment or an open\
parallel (Section 4).

The destination chain `Cn` of a closed parallel is not an entry: it receives the\
mix, not connections from outside the parallel.

A prefix (`@` / `$`) does not affect `first()`.

**Example:**

```
((123)45)=
first((123))     = {1}
first(((123)45)) = {1}
```

### 6.1 Group as Destination

Fragments (4.5); `…` marks where the glyph continues.

```
1(234)         1 → 2,  2 → 3 → 4
1(2-3-4-)      1 → 2,  1 → 3,  1 → 4,  mix(2,3,4)
1(2-3-4)       1 → 2,  1 → 3,  mix(2,3) → 4
1(2-3-4=5=)    1 → 2,  1 → 3,  1 → 5 → output,  mix(2,3) → 4 → output
1(23=45=)      1 → 2,  1 → 4,  2 → 3 → output,  4 → 5 → output
1(23-4-5)      1 → 2,  1 → 4,  2 → 3,  mix(3,4) → 5
```

To connect to the head of one branch only, place the source in that branch's\
chain instead of in front of the group:

```
21-3-4=        2 → 1,  mix(1,3) → 4 → output
```

---

## 7. Definition of `exits()`

`exits(X)` is the set of output nodes that `X` provides to whatever follows it.

**Recursive definition:**

| X                          | exits(X)                                |
| -------------------------- | --------------------------------------- |
| operator                   | { that operator }                       |
| group `( expr )`           | `exits(expr)`                           |
| chain `A B C`              | `exits(C)`  (the last atom)             |
| closed parallel `C1-…-Cn`  | `exits(Cn)`  (the destination chain; n ≥ 3) |
| open parallel `C1-…-Cn-`   | `exits(C1) ∪ … ∪ exits(Cn)`  (n ≥ 2)        |
| expr `S1 S2 … Sn`          | `exits(S1) ∪ … ∪ exits(Sn)`                 |

As in Section 6, `Sn` may be an open parallel.

A trailing `=` does not change `exits()`; it adds an edge to the device output.

### 7.1 Series Group

```
(654)
exits = {4}
```

### 7.2 Completed Group

```
(65=4=)
exits = {5, 4}
```

Two closed chains; the union of their exits.

### 7.3 Open Group

```
(65-4-)
exits = {5, 4}
```

Open parallel of the chains `65` and `4`; the union of their exits.

### 7.4 Nested Group

```
(123)
exits = {3}

((123)45)
exits = {5}
```

---

## 8. Nested Group Semantics

### 8.1 Principles

* `first()` is determined recursively
* `exits()` returns the set of final outputs

### 8.2 Example

```
((123)45)=
```

Equivalent:

```
12345=
```

```
first = {1}
exits = {5}
```

---

## 9. Feedback Semantics

### 9.1 Basic Rule

```
@(group)
```

* Adds feedback edges
* `mix(exits(group)) → first(group)`, to each element of `first(group)`
* The normal forward flow of the group is preserved

Fragments (4.5):

```
@(234)         2 → 3 → 4,  4 → 2
@(2-3-4-)      mix(2,3,4) → 2,  mix(2,3,4) → 3,  mix(2,3,4) → 4
@(2-3-4)       mix(2,3) → 4,  4 → 2,  4 → 3
@(2-3-4=5=)    mix(2,3) → 4 → output,  5 → output,
               mix(4,5) → 2,  mix(4,5) → 3,  mix(4,5) → 5
@(23=45=)      2 → 3 → output,  4 → 5 → output,
               mix(3,5) → 2,  mix(3,5) → 4
@(23-4-5)      2 → 3,  mix(3,4) → 5,  5 → 2,  5 → 4
```

### 9.2 Important Principle

Feedback does not modify the structure. It only adds edges.

### 9.3 Example

```
(@(12)3)=4=
```

**Interpretation:**

```
(12):
  1 → 2

@(12):
  1 → 2 → 1   (feedback)

forward:
  2 → 3 → output
  4 → output

first((@(12)3)=) = {1}
exits((@(12)3)=) = {3}
```

Equivalent:

```
@(12)3=4=
```

### 9.4 Feedback Target (`$`)

`$` is a symbol used inside `@(group)` to explicitly specify the destination\
operator of feedback edges.

```
$A
```

* `$A` indicates that operator `A` is the feedback destination
* If `$` is not present, the feedback returns to `first(group)` (default behavior)
* Multiple `$` targets may be specified; in that case, `exits(group)` feeds back\
  to all specified targets
* `$` may appear anywhere inside `@(group)`
* `$` belongs to the nearest enclosing `@(group)`. When `@(group)` is nested, a `$`\
  inside the inner `@(group)` sets a target for the inner group only
* `$` applied to a group targets every element of `first(group)` (Section 6), so\
  `$(12) ≡ $12`

`$` on a group, shown as fragments (4.5) inside an enclosing `@(group)`:

```
$(234)         ≡ $234          target: 2
$(2-3-4-)      ≡ $2-$3-$4-     targets: 2, 3, 4
$(2-3-4)       ≡ $2-$3-4       targets: 2, 3
$(2-3-4=5=)    ≡ $2-$3-4=$5=   targets: 2, 3, 5
$(23=45=)      ≡ $23=$45=      targets: 2, 4
$(23-4-5)      ≡ $23-$4-5      targets: 2, 4
```

**Example:**

```
@($12$3)4=
  exits(group) = {3}
  3 → 1   (feedback: exits → $1)
  3 → 3   (feedback: exits → $3, self feedback)
  3 → 4 → output

@(1-$2-)3=
  exits(group) = {1, 2}
  mix(1,2) → 2  (feedback: exits → $2)
  mix(1,2) → 3 → output
```

```
@(1-2-)3=
  exits(group) = {1, 2}
  mix(1,2) → 1  (no $: feedback returns to first(group) = {1, 2})
  mix(1,2) → 2
  mix(1,2) → 3 → output
```

Nested `@(group)`:

```
@(@(1$2)3)4=
  inner @(1$2):  exits = {2}
    2 → 2        ($2 belongs to the inner group)
  outer @(…3):   exits = {3}
    3 → 1        (no $ of its own: returns to first(group) = {1})
  1 → 2 → 3 → 4 → output
```

**Omission rule for `$`:**

If the `$` targets are exactly the elements of `first(group)`, they may be omitted.

```
@($123)4=    ≡  @(123)4=
@($1-$2-)3=  ≡  @(1-2-)3=
@($1)2=      ≡  @(1)2=   ≡  @12=   (canonical form)
```

`@($1-2-)3=` and `@(1-2-)3=` are not equivalent: `first(group) = {1, 2}`, and the\
explicit `$1` excludes `2`.

### 9.5 Relationship between `@` and `$` inside `@(group)`

`@` applied to an operator is self feedback (5.5), inside `@(group)` as\
everywhere else. It is not a feedback target, and it does not count as a `$` when\
deciding whether the default target `first(group)` applies.

When `exits(group)` consists of that operator alone, `@A` and `$A` add the same\
edge `A → A`:

```
@($12$3)4=  ≡  @($12@3)4=      exits(group) = {3}; both add 3 → 3
```

`$` notation is recommended and considered canonical.

In other cases the two differ:

```
@($1-2-$3-)4=  ≢  @($1-2-@3-)4=
```

```
@($1-2-$3-)4=
  mix(1,2,3) → 1   ($1: exits → 1)
  mix(1,2,3) → 3   ($3: exits → 3)
  mix(1,2,3) → 4 → output

@($1-2-@3-)4=
  mix(1,2,3) → 1   ($1: exits → 1)
  3 → 3            (@3: self feedback)
  mix(1,2,3) → 4 → output
```

```
@(12@3)4=
  1 → 2 → 3 → 4 → output
  3 → 3            (@3: self feedback)
  3 → 1            (no $: feedback returns to first(group) = {1})
```

---

## 10. Open Parallel Equivalence

```
(A-B-C-)D  ≡  A-B-C-D
```

Left side: the group's `exits` are `{A,B,C}`, and the group is in series with\
`D`, so `mix(A,B,C) → first(D)`.\
Right side: a closed parallel with destination chain `D`, so `mix(A,B,C) → D`.

---

## 11. Equivalence of Expressions

The following represent the same structure:

```
(@1-2-3-4)=
(@1-2-3-4=)
(@1-2-3-)4=
@1-2-3-4=
```

**Interpretation:**

```
1 → 1
mix(1, 2, 3) → 4 → output
```

---

## 12. Canonical Form

Canonical form is a recommended representation for expressing algorithm lists\
within a device in a unique and stable manner.

The following priorities are used:

| Priority | Principle                                         | Description                                                              |
| -------- | ------------------------------------------------- | ------------------------------------------------------------------------ |
| 1        | Structural correctness                            | Mandatory: the topology must be correctly represented                    |
| 2        | Unified ordering (primary rule)                   | Use either ascending or descending order consistently within a device    |
| 3        | Fewer occurrences of the same operator            | Prefer the representation in which each operator is written fewer times  |
| 4        | Elimination of redundant symbols                  | Remove unnecessary `$`, grouping, `=` symbols for minimal representation |

**Example — occurrences (priority 3):**

```
@12=@13=@14=    1 written three times
@1(2=3=4=)      each operator written once   ← canonical
```

```
@124=34=        4 written twice
@12-3-4=        each operator written once   ← canonical
```

Priority 3 applies even when the result is longer:

```
@65=@64=321=    6 written twice      (12 characters)
@6(5=4=)=321=   each written once    (13 characters)   ← canonical
```

**Example — ordering before length (priority 2):**

```
21=@6(5=4=3=)    13 characters, segments not in descending order
@6(5=4=3=)=21=   14 characters, descending              ← canonical (descending device)
```

**Example — `$` (priority 4), shown as fragments (4.5):**

For the topology:

```
mix(1,2) → 2
```

```
@(1-$2-)   ← ascending
@($2-1-)   ← descending
```

Both require `$`, since `first(@(…))` is `{1, 2}` without it (Section 6). The two\
are equally short; the device's ordering decides.

If the device adopts ascending order, the canonical form is:

```
@(1-$2-)
```

If the device adopts descending order, the canonical form is:

```
@($2-1-)
```

For the topology `mix(1,2) → 1, mix(1,2) → 2`, `$` is redundant and is removed\
(priority 4): `@(1-2-)` (ascending) or `@(2-1-)` (descending).

**Recommended example (ascending device):**

```
@1-2-3-4=
```

In addition to structural correctness, canonical form is also intended to improve visual clarity.

OTG expressions are designed to be read directly by humans.\
Therefore, canonicalization should prefer representations that are:

* shorter
* visually simpler
* easier to interpret at a glance

This includes:

* unifying operator ordering (ascending or descending)
* writing each operator as few times as possible
* minimizing redundant symbols
* avoiding unnecessary grouping

The goal is not only uniqueness, but also readability.

---

## 13. Constraints

* Must be within `glyph_length`
* A space may appear only as trailing padding (2.1)
* Open parallel cannot stand alone (incomplete)
* Groups must be syntactically closed
* A glyph must end closed (4.5)
* Within an `<expr>`, every segment except the last must be closed by `=`
* `=` may appear only after the last chain of a segment; it cannot follow an open\
  parallel (4.2)
* `$` is valid only inside `@(group)`
* `@` and `$` are mutually exclusive on the same atom
* `$` belongs to the nearest enclosing `@(group)` (9.4)

---

## 14. Design Characteristics

* Representable using ASCII only
* Independent of operator count
* Supports DX7 / YM2151 / YM2203
* Convertible to AST / graph structures

---

## 15. Formal Summary

```
series:      A B         ⇒ exits(A) → each of first(B)
parallel:    C1-…-Cn     ⇒ mix(exits(C1..Cn-1)) → each of first(Cn)   (n ≥ 3)
open:        C1-…-Cn-    ⇒ mix(exits(C1..Cn))                         (n ≥ 2)
terminator:  A=          ⇒ exits(A) → output, segment closed
feedback:    @A          ⇒ A → A
group fb:    @(G)        ⇒ mix(exits(G)) → each of first(G)
fb target:   @($A…G)     ⇒ mix(exits(G)) → each $A
identity:    A … A       ⇒ the same node
```

---

## 16. Extended Operator Identifier

If the number of operators increases beyond single-character representation, or\
if arbitrary names are required, `[ ]` may be used.

### Syntax

```
<long_operator> ::= "[" <identifier> "]"
<identifier>    ::= <id_char> { <id_char> }
<id_char>       ::= [0-9A-Za-z_]
```

### Examples

```
OP10     → [10]
OP99     → [99]
OP300    → [300]
OP alpha → [alpha]
OP beta  → [beta]
```

```
@[10]987654321=
@[alpha][beta]=
```

---

## 17. Recommended Operator Numbering

Operator numbering should follow the conventions used in the target FM device datasheet.

### Examples

```
YM2151 : @1234=
DX7    : @6543=21=
```

### 17.1 Rationale

* Consistency with existing documentation
* Alignment with hardware understanding
* Reduced cognitive load for users

### 17.2 Note

This specification preserves freedom of notation. Any representation that can be\
interpreted as equivalent is allowed.

**Example:**

For DX7 alg1:

```
A = OP6
B = OP5
C = OP4
D = OP3
E = OP2
F = OP1
```

The following representation is also valid (though not recommended):

```
alg1: @ABCD=EF=
```

Similarly, named identifiers using `[ ]` are allowed:

```
[alpha] = OP1
[beta]  = OP2

@[alpha][beta]=
```

---

## 18. Recommended Ordering Policy

It is recommended to use a consistent ordering (ascending or descending) for\
operator numbering across all algorithms within the same device.

By combining the rules in Sections 17 and 18, a unique recommended representation\
can be determined for any FM device.

### Principles

* Sections 17 and 18 are recommendations; all equivalent expressions remain valid
* However, following the recommended notation ensures a unique canonical form for\
  the device

### Example

For YM2151 alg7, the following are all equivalent:

```
@1=2=3=4=
4=3=2=@1=
4=3=@1=2=
2=4=3=@1=
```

Since alg0 uses ascending order (`@1234=`), the recommended representation for\
alg7 is:

```
alg7: @1=2=3=4=
```

---

## 19. Implementation Notes

* tokenizer: `$` and bracket support required
* parser: recursive descent recommended
* parse order: `expr → segment → parallel → chain → atom → group`
* a chain is read greedily until `=`, `-`, `)`, or end of glyph
* AST structure: node / operator / group / chain / parallel / feedback_target
* graph construction: edge-based, with edges held in a set
* operator identity: resolve identifiers to a node table, not to occurrences
* recommended validation: parse every entry in the device tables linked from\
  Section 22 and compare the resulting edge sets

---

## 20. Future Extensions

* algorithm_family
* operator_count
* metadata linkage
* OED integration

---

## 21. Versioning

### Current Version

draft version 0.03

### Change Log

| Date       | Version | Changes |
| ---------- | ------- | ------- |
| 2026-03-20 | 0.00    | Initial version |
| 2026-03-23 | 0.01    | Added `$` (feedback target), defined exclusivity between `@` and `$`, defined the relationship between `@` and `$` within `@(group)`, redefined canonical form priorities, added `a-z` and `0` to short_operator |
| 2026-08-03 | 0.02    | Grammar corrected to match the notation in use. Series binds tighter than parallel (4.1). `=` defined as chain terminator (4.2). Segment closure rule added (4.3). Parallel operands generalised to chains (5.3). Operator identity defined (3.2). Edge set semantics defined (5.6). `first()` / `exits()` generalised beyond groups (6, 7). `$` target validity constrained (13). Verification material added (22), with device tables moved to `examples/`. **The notation itself is unchanged.** |
| 2026-11-18 | 0.03    | `=` moved from `<chain>` to the segment level; `=` inside a parallel and after an open parallel rejected (4, 4.2, 5.2). Segment sequencing example corrected (4.3). `$` inside nested `@(group)` bound to the nearest enclosing `@(group)` (9.4, 13). `$` on a group defined as targeting `first(group)` (9.4). `@` on an operator inside `@(group)` defined as self feedback in all cases; the rule that read it as `$` when the operator is in `exits(group)` removed, since it contradicted the example `@($1-2-$3-)4= ≢ @($1-2-@3-)4=` (9.5). Identifier case sensitivity stated; single-character long operator identified with the short operator (3.2). Space restricted to trailing padding (2.1, 13). `=` described as segment terminator (2, 3.3, 4.1, 22.2). Section 22.2 restructured: vector count corrected; `first()` redefined as a set: closed parallel → entries of `C1…Cn-1`, open parallel → entries of all chains, expr → entries of all segments; applied to series destinations, `$` on a group, and the default target of `@(group)` (5, 6, 6.1, 9.1, 9.4); `$` omission rule restated for sets (9.4); canonical form example rewritten (12). Glyph closure required; fragments defined (4.5, 13). Canonical form priority added: fewer occurrences of the same operator (12); OPM 5 and DX7 19–25 rewritten accordingly in `examples/`. Redundant `$` target constraint replaced by the nearest-group rule (13). Vectors for 0.03 and rejection vectors added (22.2). Closed parallel requires at least two branches; `1-2=` rejected (4, 5.3); minimum counts stated for parallels (5.4, 6, 7, 15); `Sn` of an expr may be an open parallel (6, 7); Formal Summary parallel line aligned to sets (15). **Glyphs in the device tables keep their meaning. Glyphs that use a parallel or a multi-segment group as a destination or feedback target change meaning (see Note on 0.02).** |

### Note on 0.01

Draft 0.01 is superseded. Its Sections 4, 4.1, 5.3, 6 and 7 do not correctly\
describe the notation as used, and a parser built from them will misparse glyphs\
of the form `A-BC-D=` and `…=…=`. The reasoning behind each 0.02 change is\
recorded separately in the OTG rationale document.

### Note on 0.02

Draft 0.02 is superseded. Its grammar admitted `=` on any chain, including chains\
inside a parallel (`1-2=-3=`), without stating a meaning for that form. No glyph in\
the device tables uses it.

Its Section 9.5 stated that `@` on an operator in `exits(group)` is read as `$`,\
and in the same section gave `@($1-2-$3-)4= ≢ @($1-2-@3-)4=`, where `3` is in\
`exits(group)`. The two cannot both hold. 0.03 keeps the example and removes the\
rule. Glyphs in which `@` is applied to an operator in `exits(group)` of an\
`@(group)`, and in which that `exits(group)` has more than one operator or no `$`\
appears, read differently under the removed rule. No glyph in the device tables is\
of this form.

Its Section 6 defined `first()` of a parallel as `first(C1)` and of an expr as\
`first(S1)`. 0.03 defines both as sets of all entries. Under 0.02,\
`5(1-2-)` connected `5 → 1` only, and `@(1-2-)` fed back to `1` only; under 0.03\
they connect to `1` and `2`. The 0.02 reading remains expressible: place the\
source in the first branch's chain (`51-2-`), or name the target with `$`\
(`@($1-2-)`). No glyph in the device tables uses a parallel or a multi-segment\
group as a destination or feedback target.

Its grammar allowed a glyph to end with an open chain, as in `@134=23`. 0.03\
requires a glyph to end closed (4.5). Examples in this specification that end\
open are fragments.

---

## 22. Verification Material

The material in this section is informative. It is provided as reference and as\
regression input; it does not define the language.

### 22.1 Reference Tables

Device algorithm tables are maintained separately, so that a single copy exists:

* [YM2151 / YM2203 — 8 algorithms](../examples/ym2151_ym2203.md)
* [DX7 — 32 algorithms](../examples/dx7.md)

Each table lists the glyph and the structure produced by the rules in\
Sections 4–9. An implementation can be checked by parsing every entry and\
comparing the resulting edge set against the structure column.

These tables are not normative. OTG is device-independent, whereas the tables are\
not; binding the language to two specific devices would make every future device\
a specification amendment. The tables verify this specification. They do not\
constitute it.

### 22.2 Selected Vectors

The following entries each isolate a rule that is easy to get wrong.

The device tables exercise only part of 0.03: the canonical forms of OPM 5 and\
DX7 19–25 require `first()` of a group to be the union over its segments. No table\
entry uses a parallel as a destination or feedback target, nests `@(group)`, or\
places `@` or `$` on members of a group's exits. Vectors 22.2.2 and 22.2.3 cover\
these.

#### 22.2.1 Rules from 0.02

**OPM 2 — `@1-23-4=`**\
Requires that `-` take chains as operands (4.1). The `23` between the two `-`\
must be read as one branch. Reading it as separate items yields a plain series,\
which is a different algorithm.

**DX7 7 — `@65-4-3=21=`**\
Requires that `=` terminate the segment (4.2). Without termination, `21=` continues\
the parallel instead of beginning a new segment.

**DX7 16 — `@65-43-2-1=`**\
Requires both of the above at once. Two multi-operator branches, a closed\
parallel, and a terminating `=`.

**`@12=@13=@14=`** (OPM 5, written without grouping)\
Requires operator identity across occurrences (3.2) and edge set semantics (5.6).\
The three occurrences of `1` are one node, and the feedback `1→1` — declared\
three times — is applied once. The table's canonical form `@1(2=3=4=)` does not\
repeat `1`, so this form is listed here separately.

**DX7 3 vs 4 — `@654=321=` and `@(654)=321=`**\
Distinguishes self feedback (`6→6`) from group feedback (`4→6`). The two glyphs\
differ by one pair of parentheses and describe different topologies.

#### 22.2.2 Rules from 0.03

| Glyph | Expected | Isolates |
| ----- | -------- | -------- |
| `@1(2=3=4=)` | 1→1; 1→2→out; 1→3→out; 1→4→out | `first()` of an expr is the union over segments (6) |
| `1(2-3-4=5=)=` | 1→2; 1→3; 1→5; mix(2,3)→4→out; 5→out | closed parallel entries exclude `Cn`; union with a further segment (6) |
| `@(1-2-)3=` | mix(1,2)→1; mix(1,2)→2; mix(1,2)→3→out | default feedback target is the set `first(group)` (9.1) |
| `@($1-2-@3-)4=` | mix(1,2,3)→1; 3→3; mix(1,2,3)→4→out | `@` on an operator is self feedback inside `@(group)` (9.5) |
| `@(@(1$2)3)4=` | 2→2; 3→1; 1→2→3→4→out | `$` belongs to the nearest enclosing `@(group)` (9.4) |
| `@($(1-2-)3)4=` | mix(1,2)→3; 3→1; 3→2; 3→4→out | `$` on a group targets every element of `first(group)` (9.4) |

#### 22.2.3 Rejection

| Glyph | Reason |
| ----- | ------ |
| `1-2=-3=` | `=` inside a parallel (4.2); equally, `1-2=` is a closed parallel with one branch (5.3) |
| `1-2-3=-4=` | `-` after `=` (4.2) |
| `1-2-=` | `=` after an open parallel (4.2) |
| `1-2=` | closed parallel with one branch (5.3) |
| `@134=23` | glyph does not end closed (4.5) |
| `@12= 34=` | space inside the glyph (2.1) |
| `$12=` | `$` outside `@(group)` (9.4) |
