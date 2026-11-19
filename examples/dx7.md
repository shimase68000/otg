# DX7 Algorithm Table

**OED Operator Topology Glyph (connection_glyph)**

**Date Created:** 2026-03-23\
**Last Updated:** 2026-11-18\
**Author:** UG.\
**Covers:** OTG Syntax Specification draft 0.03

---

## Device

| Item       | Value                       |
| ---------- | --------------------------- |
| Device     | Yamaha DX7                  |
| Operators  | 6                           |
| Algorithms | 32                          |
| Notation   | OED Operator Topology Glyph |

---

## Algorithm List

|  # | connection_glyph   | structure |
| -: | ------------------ | --------- |
|  1 | `@6543=21=`        | 6→6; 6→5→4→3→out; 2→1→out |
|  2 | `6543=@21=`        | 2→2; 6→5→4→3→out; 2→1→out |
|  3 | `@654=321=`        | 6→6; 6→5→4→out; 3→2→1→out |
|  4 | `@(654)=321=`      | 4→6; 6→5→4→out; 3→2→1→out |
|  5 | `@65=43=21=`       | 6→6; 6→5→out; 4→3→out; 2→1→out |
|  6 | `@(65)=43=21=`     | 5→6; 6→5→out; 4→3→out; 2→1→out |
|  7 | `@65-4-3=21=`      | 6→6; 6→5; mix(5,4)→3→out; 2→1→out |
|  8 | `65-@4-3=21=`      | 4→4; 6→5; mix(5,4)→3→out; 2→1→out |
|  9 | `65-4-3=@21=`      | 2→2; 6→5; mix(5,4)→3→out; 2→1→out |
| 10 | `6-5-4=@321=`      | 3→3; mix(6,5)→4→out; 3→2→1→out |
| 11 | `@6-5-4=321=`      | 6→6; mix(6,5)→4→out; 3→2→1→out |
| 12 | `6-5-4-3=@21=`     | 2→2; mix(6,5,4)→3→out; 2→1→out |
| 13 | `@6-5-4-3=21=`     | 6→6; mix(6,5,4)→3→out; 2→1→out |
| 14 | `@6-5-43=21=`      | 6→6; mix(6,5)→4→3→out; 2→1→out |
| 15 | `6-5-43=@21=`      | 2→2; mix(6,5)→4→3→out; 2→1→out |
| 16 | `@65-43-2-1=`      | 6→6; 6→5; 4→3; mix(5,3,2)→1→out |
| 17 | `65-43-@2-1=`      | 2→2; 6→5; 4→3; mix(5,3,2)→1→out |
| 18 | `654-@3-2-1=`      | 3→3; 6→5→4; mix(4,3,2)→1→out |
| 19 | `@6(5=4=)=321=`    | 6→6; 6→5→out; 6→4→out; 3→2→1→out |
| 20 | `6-5-4=@3(2=1=)`   | 3→3; mix(6,5)→4→out; 3→2→out; 3→1→out |
| 21 | `6(5=4=)=@3(2=1=)` | 3→3; 6→5→out; 6→4→out; 3→2→out; 3→1→out |
| 22 | `@6(5=4=3=)=21=`   | 6→6; 6→5→out; 6→4→out; 6→3→out; 2→1→out |
| 23 | `@6(5=4=)=32=1=`   | 6→6; 6→5→out; 6→4→out; 3→2→out; 1→out |
| 24 | `@6(5=4=3=)=2=1=`  | 6→6; 6→5→out; 6→4→out; 6→3→out; 2→out; 1→out |
| 25 | `@6(5=4=)=3=2=1=`  | 6→6; 6→5→out; 6→4→out; 3→out; 2→out; 1→out |
| 26 | `@6-5-4=32=1=`     | 6→6; mix(6,5)→4→out; 3→2→out; 1→out |
| 27 | `6-5-4=@32=1=`     | 3→3; mix(6,5)→4→out; 3→2→out; 1→out |
| 28 | `6=@543=21=`       | 5→5; 6→out; 5→4→3→out; 2→1→out |
| 29 | `@65=43=2=1=`      | 6→6; 6→5→out; 4→3→out; 2→out; 1→out |
| 30 | `6=@543=2=1=`      | 5→5; 6→out; 5→4→3→out; 2→out; 1→out |
| 31 | `@65=4=3=2=1=`     | 6→6; 6→5→out; 4→out; 3→out; 2→out; 1→out |
| 32 | `@6=5=4=3=2=1=`    | 6→6; 6→out; 5→out; 4→out; 3→out; 2→out; 1→out |

### Notation used in the structure column

* `A→B` — connection edge
* `A→out` — connection to the device output
* `A→A` — self feedback
* `A→B` where `A` is downstream of `B` — group feedback (see #4, #6)
* `mix(…)→X` — the listed nodes are summed into `X`

---

## Notes

* Operator numbering follows the DX7 datasheet convention\
  (see Specification, Section 17)
* Operator order is unified in descending order (see Specification, Section 18)
* The structure column is unchanged from draft 0.02
* Algorithms 19–25 are written in the canonical form of draft 0.03\
  (Specification, Section 12). Their draft 0.02 glyphs remain valid and\
  equivalent (table below)
* The glyphs for algorithms 19–25 depend on `first()` of a group being the union\
  over its segments (draft 0.03). Under draft 0.02 they parse to different\
  structures.

|  # | draft 0.02 glyph   |
| -: | ------------------ |
| 19 | `@65=@64=321=`     |
| 20 | `6-5-4=@32=@31=`   |
| 21 | `65=64=@32=@31=`   |
| 22 | `@65=@64=@63=21=`  |
| 23 | `@65=@64=32=1=`    |
| 24 | `@65=@64=@63=2=1=` |
| 25 | `@65=@64=3=2=1=`   |

---

## Use as Regression Input

This table is informative — it does not define the language. It is, however, the\
material the specification was verified against, and it is suitable as regression\
input for an implementation: parse each glyph and compare the resulting edge set\
against the structure column.

Algorithm 7 (`@65-4-3=21=`) requires that `=` terminate a segment, so that `21=`\
begins a new segment rather than continuing the parallel.\
Algorithm 16 (`@65-43-2-1=`) requires that `-` take chains as operands, and that\
`=` terminate a segment, simultaneously.\
Algorithms 3 and 4 (`@654=321=` / `@(654)=321=`) distinguish self feedback from\
group feedback.\
Algorithms 19–25 require `first()` of a group to be the union over its segments.

See Specification Section 22 for details.
