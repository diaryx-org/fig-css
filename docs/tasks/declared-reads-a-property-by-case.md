---
title: A rule's declared reads a property by its case
description: '`Rule::declared` compares a property name byte for byte, so `FILL: red` is not found under `fill`, and in `fill: red; FILL: blue` it answers the value that does not apply'
status: done
created: 2026-11-05
updated: 2026-11-05
part_of: "[Tasks](tasks.md)"
---

# A rule's declared reads a property by its case

**Status.** Done, by `fix: a rule's declared reads a property in any case`:
`Rule::declared` matches through `same_property`, and
`a_rule_reads_a_property_in_any_case` in `src/lib.rs` holds the repro, a
custom property staying case-sensitive, and the value and the
`Declaration`'s `property` staying as written.

CSS compares a property name without regard to ASCII case, except a custom
property's (`--brand`). `same_property` says so, and `declaration` and
`with_declaration` read a declaration list through it. `Rule::declared`, the
same question asked of a stylesheet's rule, compares with `==`, so the two
answer differently for one set of declarations.

## Repro

On `main` (`4d83f31`):

```rust
use fig_css::{declaration, stylesheet};

// A style attribute reads the property in any case, and takes the last.
assert_eq!(declaration("FILL: red", "fill").as_deref(), Some("red"));
assert_eq!(declaration("fill: red; FILL: blue", "fill").as_deref(), Some("blue"));

// A rule's block does neither.
let rule = |css| stylesheet(css).unwrap().remove(0);
assert_eq!(rule("rect { FILL: red }").declared("fill"), None);
assert_eq!(rule("rect { fill: red; FILL: blue }").declared("fill"), Some("red"));
```

The second is the worse of the two: `declared` promises the last repeat,
"since that is the one that applies", and answers the one a browser overrides.

## What it costs

thorn's `crates/thorn-svg-core/src/style.rs` asks `declared("stroke")` and
`declared("fill")` of each rule, so a hand-written or imported sheet that
spells either in another case reads to it as a rule without that property.
Nothing thorn writes spells them so.

## Done when

`Rule::declared` matches through `same_property`, the repro's two
`declared` calls answer `Some("red")` and `Some("blue")`, a custom property
stays case-sensitive (`--Brand` is not found under `--brand`), and a test
holds it. The value and the `Declaration`'s `property` stay as written: only
the comparison changes.
