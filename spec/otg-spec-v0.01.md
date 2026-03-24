# OED Operator Topology Glyph (connection_glyph)

**Syntax Specification — draft version 0.01**

**Last Updated:** 2026-03-23
**Author:** UG.

---

## 1. Overview

`connection_glyph` is a notation for representing operator connection structures (algorithms) of FM synthesis using compact ASCII strings.

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
=               carrier (final output)
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

### 3.2 Symbols

```
@    feedback
=    carrier
-    parallel
( )  grouping
[ ]  multi-digit / named operator identifier
$    feedback target
```

---

## 4. Grammar (Informal BNF)

```
<glyph>           ::= <expr>
<expr>            ::= <series>
<series>          ::= <item> { <item> }
<item>            ::= <parallel> | <atom>
<parallel>        ::= <atom> "-" <atom> { "-" <atom> }
                    | <atom> "-" <atom> { "-" <atom> } "-"   ; open parallel
<atom>            ::= ( <feedback> | <feedback_target> )? ( <operator> | <group> ) <carrier>?
<feedback>        ::= "@"
<carrier>         ::= "="
<group>           ::= "(" <expr> ")"
<feedback_target> ::= "$" <operator>
```

### 4.1 Operator Precedence (Important)

| Priority | Element                    | Symbol          |
| -------- | -------------------------- | --------------- |
| 1        | group                      | `( )`           |
| 2        | feedback / feedback target | `@` / `$`       |
| 3        | parallel                   | `-`             |
| 4        | series                     | (concatenation) |
| 5        | carrier                    | `=`             |

> `@` and `$` are mutually exclusive and cannot coexist on the same atom.

### 4.2 Feedback Binding Rule

`@` applies only to the immediately following element (operator or group).

```
@12  ≡  (@1)2
```

---

## 5. Core Semantics

### 5.1 Series

```
A B
```

```
A → B
```

Example:

```
1234=
1 → 2 → 3 → 4 → output
```

### 5.2 Carrier

```
A=
```

```
A → output
```

### 5.3 Parallel (Closed)

```
A-B-C-D
```

```
mix(A, B, C) → D
```

### 5.4 Parallel (Open)

```
A-B-C-
```

```
mix(A, B, C)
```

> A subsequent connection is required.

### 5.5 Self Feedback

```
@A
```

```
A → A
```

### 5.6 Group

```
(expr)
```

A group forms a syntactic scope and serves as the evaluation unit for `first()` and `exits()`.

### 5.7 Group Feedback

```
@(group)
```

```
mix(exits(group)) → first(group)
```

---

## 6. Definition of `first(group)`

`first(group)` is the first operator processed within the group.

**Recursive definition:**

* If the first element is an operator → return that operator
* If the first element is a group → return its `first()`

**Example:**

```
((123)45)=
first((123))     = 1
first(((123)45)) = 1
```

---

## 7. Definition of `exits(group)`

`exits(group)` is the set of output nodes that the group provides to the outside.

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

### 7.3 Open Group

```
(65-4-)
exits = {5, 4}
```

### 7.4 Nested Group

```
(123)
exits = {3}

((123)45)
exits = {5}
```

**Recursive rule:**
The `exits` of subgroups are resolved internally, and only the final outputs of the group are exposed externally.

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

`$` is a symbol used inside `@(group)` to explicitly specify the destination operator of feedback edges.

```
$A
```

* `$A` indicates that operator `A` is the feedback destination
* If `$` is not present, the feedback returns to `first(group)` (default behavior)
* Multiple `$` targets may be specified; in that case, `exits(group)` feeds back to all specified targets
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

Within `@(group)`, if `@` is applied to an operator that belongs to `exits(group)`, it is interpreted as equivalent to `$`.

```
@($12$3)4=  ≡  @($12@3)4=
```

However, `$` notation is recommended and considered canonical.

If `@` is applied to an operator that is not in `exits(group)`, it is interpreted as self feedback.

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

Canonical form is a recommended representation for expressing algorithm lists within a device in a unique and stable manner.

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

OTG expressions are designed to be read directly by humans.
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
* `$` is valid only inside `@(group)`
* `@` and `$` are mutually exclusive on the same atom

---

## 14. Design Characteristics

* Representable using ASCII only
* Independent of operator count
* Supports DX7 / YM2151 / YM2203
* Convertible to AST / graph structures

---

## 15. Formal Summary

```
series:      A B         ⇒ A → B
parallel:    A-B-C-D     ⇒ mix(A,B,C) → D
open:        A-B-C-      ⇒ mix(A,B,C)
carrier:     A=          ⇒ A → output
feedback:    @A          ⇒ A → A
group fb:    @(G)        ⇒ mix(exits(G)) → first(G)
fb target:   @($A...G)   ⇒ mix(exits(G)) → each $A
```

---

## 16. Extended Operator Identifier

If the number of operators increases beyond single-character representation, or if arbitrary names are required, `[ ]` may be used.

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

This specification preserves freedom of notation. Any representation that can be interpreted as equivalent is allowed.

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

It is recommended to use a consistent ordering (ascending or descending) for operator numbering across all algorithms within the same device.

By combining the rules in Sections 17 and 18, a unique recommended representation can be determined for any FM device.

### Principles

* Sections 17 and 18 are recommendations; all equivalent expressions remain valid
* However, following the recommended notation ensures a unique canonical form for the device

### Example

For YM2151 alg7, the following are all equivalent:

```
@1=2=3=4=
4=3=2=@1=
4=3=@1=2=
2=4=3=@1=
```

Since alg0 uses ascending order (`@1234=`), the recommended representation for alg7 is:

```
alg7: @1=2=3=4=
```

---

## 19. Implementation Notes

* tokenizer: `$` and bracket support required
* parser: recursive descent recommended
* AST structure: node / operator / group / parallel / feedback_target
* graph construction: edge-based approach

---

## 20. Future Extensions

* algorithm_family
* operator_count
* metadata linkage
* OED integration

---

## 21. Versioning

### Current Version

draft version 0.01

### Change Log

| Date       | Version | Changes                                                                                                                                                                                                             |
| ---------- | ------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 2026-03-20 | 0.00    | Initial version                                                                                                                                                                                                     |
| 2026-03-23 | 0.01    | Added `$` (feedback target), defined exclusivity between `@` and `$`, defined the relationship between `@` and `$` within `@(group)`,<br>redefined canonical form priorities, added `a-z` and `0` to short_operator |
