# OED Operator Topology Glyph (connection_glyph)

**Syntax Specification — draft version 0.02**

**Last Updated:** 2026-08-03\
**Author:** UG.

> **Note on this revision**
> Version 0.02 does not change the notation. It corrects the written grammar so
> that it matches the notation that was already in use in the YM2151 / YM2203 and
> DX7 algorithm tables (2026-03-23). Every glyph listed in those tables parses
> correctly under this specification; several of them did not parse correctly
> under draft 0.01. See Section 21 for the change log and Section 22 for the
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
=               chain terminator / output
-               parallel connection
( )             grouping
[ ]             multi-digit / named operator identifier
$               feedback target
space           padding (used only for storage)
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

`[10]` and `10` are distinct identifiers. `[1]` and `1` denote the same operator.

### 3.3 Symbols

```
@    feedback
=    chain terminator / output
-    parallel
( )  grouping
[ ]  multi-digit / named operator identifier
$    feedback target
```

---

## 4. Grammar (Informal BNF)

```
<glyph>     ::= <expr>

<expr>      ::= { <closed_segment> } [ <segment> ]

<segment>   ::= <parallel> | <chain>
<closed_segment> ::= a <segment> whose final <chain> carries "="

<parallel>  ::= <chain> "-" <chain> { "-" <chain> }        ; closed parallel
              | <chain> "-" <chain> { "-" <chain> } "-"    ; open parallel

<chain>     ::= <atom> { <atom> } [ "=" ]

<atom>      ::= [ <prefix> ] ( <operator> | <group> )
<prefix>    ::= "@" | "$"

<group>     ::= "(" <expr> ")"
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

parses as a parallel of three chains — `1`, `23`, `4=` — not as a parallel of the\
atoms `1` and `2` followed by `3` and `4=`.

`( … )` binds tighter than everything; `@` and `$` bind to the single atom that follows them.

### 4.2 Chain Termination

A `<chain>` ends at the first of:

* `=`   (consumed as part of the chain; the chain is *closed*)
* `-`   (the chain is one operand of a parallel)
* `)`   (end of enclosing group)
* end of glyph

`=` therefore serves two purposes at once: it closes the chain, and it routes the\
chain's outputs to the device output. The closing role is primary — it is what\
makes the following expression a new structure rather than a continuation.

```
@12=34=
```

```
1 → 2 → output
3 → 4 → output
```

Without the terminating role of `=`, this would read as a single chain
`1 → 2 → 3 → 4`.

### 4.3 Segment Sequencing

Within an `<expr>`, every segment except the last must be closed. Two\
unterminated structures cannot be juxtaposed, since their relationship would be undefined.

```
@12=3=4=     valid    (three closed chains)
12 34        invalid  (no relationship defined)
```

### 4.4 Prefix Binding Rule

`@` and `$` apply only to the immediately following atom.

```
@12  ≡  (@1)2
```

`@` and `$` are mutually exclusive on the same atom.

---

## 5. Core Semantics

Semantics are defined as a set of directed edges over operator nodes, plus a set\
of edges to the device output. Because edges form a *set*, declaring the same\
edge more than once has no additional effect (see 5.6).

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

and the chain is closed.

### 5.3 Parallel (Closed)

```
C1 - C2 - … - Cn
```

```
mix( exits(C1), exits(C2), …, exits(Cn-1) ) → first(Cn)
```

where each `Ci` is a chain. When every chain is a single operator this reduces to\
the familiar form:

```
A-B-C-D   ⇒   mix(A, B, C) → D
```

With multi-operator chains:

```
1-23-4=   ⇒   1 → 4,  2 → 3 → 4,  4 → output
```

### 5.4 Parallel (Open)

```
C1 - C2 - … - Cn -
```

```
mix( exits(C1), …, exits(Cn) )
```

with no destination. An open parallel is incomplete: it must be enclosed in a group\
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

`first(X)` is the node that receives incoming connections when `X` is used as a destination.

**Recursive definition:**

| X                        | first(X)                              |
| ------------------------ | ------------------------------------- |
| operator                 | that operator                         |
| group `( expr )`         | `first(expr)`                         |
| chain `A B C`            | `first(A)`                            |
| parallel `C1-…-Cn`       | `first(C1)`                           |
| expr `S1 S2 … Sn`        | `first(S1)`                           |

A prefix (`@` / `$`) does not affect `first()`.

**Example:**

```
((123)45)=
first((123))     = 1
first(((123)45)) = 1
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
| closed parallel `C1-…-Cn`  | `exits(Cn)`  (the destination chain)    |
| open parallel `C1-…-Cn-`   | `exits(C1) ∪ … ∪ exits(Cn)`             |
| expr `S1 S2 … Sn`          | `exits(S1) ∪ … ∪ exits(Sn)`             |

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
first = 1
exits = {5}
```

---

## 9. Feedback Semantics

### 9.1 Basic Rule

```
@(group)
```

* Adds feedback edges
* `mix(exits(group)) → first(group)`
* The normal forward flow of the group is preserved

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

first((@(12)3)=) = 1
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
  mix(1,2) → 1  (no $: feedback returns to first(group) = 1)
  mix(1,2) → 3 → output
```

**Omission rule for `$`:**

If `$` points to the same operator as `first(group)`, it may be omitted.

```
@($123)4=  ≡  @(123)4=
@($1)2=    ≡  @(1)2=   ≡  @12=   (canonical form)
```

### 9.5 Relationship between `@` and `$` inside `@(group)`

Within `@(group)`, if `@` is applied to an operator that belongs to\
`exits(group)`, it is interpreted as equivalent to `$`.

```
@($12$3)4=  ≡  @($12@3)4=
```

However, `$` notation is recommended and considered canonical.

If `@` is applied to an operator that is not in `exits(group)`, it is interpreted\
as self feedback.

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
| 3        | Elimination of redundant symbols (secondary rule) | Remove unnecessary `$`, grouping, `=` symbols for minimal representation |

**Example:**

For the topology:

```
mix(1,2) → 2
```

```
@(1-$2-)   ← ascending, explicit `$`
@(2-1-)    ← descending, shorter
```

If the device adopts ascending order, the canonical form is:

```
@(1-$2-)
```

If the device adopts descending order, the canonical form is:

```
@(2-1-)
```

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

* minimizing redundant symbols
* unifying operator ordering (ascending or descending)
* avoiding unnecessary grouping

The goal is not only uniqueness, but also readability.

---

## 13. Constraints

* Must be within `glyph_length`
* Open parallel cannot stand alone (incomplete)
* Groups must be syntactically closed
* Within an `<expr>`, every segment except the last must be closed by `=`
* `$` is valid only inside `@(group)`
* `@` and `$` are mutually exclusive on the same atom
* A `$` target must denote an operator that appears within the same `@(group)`

---

## 14. Design Characteristics

* Representable using ASCII only
* Independent of operator count
* Supports DX7 / YM2151 / YM2203
* Convertible to AST / graph structures

---

## 15. Formal Summary

```
series:      A B         ⇒ exits(A) → first(B)
parallel:    C1-…-Cn     ⇒ mix(exits(C1..Cn-1)) → first(Cn)
open:        C1-…-Cn-    ⇒ mix(exits(C1..Cn))
terminator:  A=          ⇒ exits(A) → output, chain closed
feedback:    @A          ⇒ A → A
group fb:    @(G)        ⇒ mix(exits(G)) → first(G)
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
* However, following the recommended notation ensures a unique canonical form for
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

draft version 0.02

### Change Log

| Date       | Version | Changes |
| ---------- | ------- | ------- |
| 2026-03-20 | 0.00    | Initial version |
| 2026-03-23 | 0.01    | Added `$` (feedback target), defined exclusivity between `@` and `$`, defined the relationship between `@` and `$` within `@(group)`, redefined canonical form priorities, added `a-z` and `0` to short_operator |
| 2026-08-03 | 0.02    | Grammar corrected to match the notation in use. Series binds tighter than parallel (4.1). `=` defined as chain terminator (4.2). Segment closure rule added (4.3). Parallel operands generalised to chains (5.3). Operator identity defined (3.2). Edge set semantics defined (5.6). `first()` / `exits()` generalised beyond groups (6, 7). `$` target validity constrained (13). Verification material added (22), with device tables moved to `examples/`. **The notation itself is unchanged.** |

### Note on 0.01

Draft 0.01 is superseded. Its Sections 4, 4.1, 5.3, 6 and 7 do not correctly\
describe the notation as used, and a parser built from them will misparse glyphs\
of the form `A-BC-D=` and `…=…=`. The reasoning behind each 0.02 change is\
recorded separately in the OTG rationale document.

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

The following entries each isolate a rule that is easy to get wrong. Any\
implementation that handles these four correctly is unlikely to be wrong
elsewhere.

**OPM 2 — `@1-23-4=`**\
Requires that `-` take chains as operands (4.1). The `23` between the two `-`\
must be read as one branch. Reading it as separate items yields a plain series,\
which is a different algorithm.

**DX7 7 — `@65-4-3=21=`**\
Requires that `=` terminate the chain (4.2). Without termination, `21=` continues\
the parallel instead of beginning a new segment.

**DX7 16 — `@65-43-2-1=`**\
Requires both of the above at once. Two multi-operator branches, a closed\
parallel, and a terminating `=`.

**OPM 5 — `@12=@13=@14=`**\
Requires operator identity across occurrences (3.2) and edge set semantics (5.6).\
The three occurrences of `1` are one node, and the feedback `1→1` — declared\
three times — is applied once.

**DX7 3 vs 4 — `@654=321=` and `@(654)=321=`**\
Distinguishes self feedback (`6→6`) from group feedback (`4→6`). The two glyphs\
differ by one pair of parentheses and describe different topologies.

