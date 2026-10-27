---
title: Setting a repeated property edits the one that does not apply
description: When a list gives a property twice, `with_declaration` sets or removes the first, and `declaration` goes on reading the last, so the edit makes no difference
status: open
created: 2026-10-26
updated: 2026-10-26
part_of: "[Tasks](tasks.md)"
---

# Setting a repeated property edits the one that does not apply

A declaration list may give a property more than once. The last one is the one
that applies, and `declaration` and `Rule::declared` read that one.
`with_declaration` edits through fig's editor, with the path
`[Segment::Key(property)]`. A path names the first repeat, as the crate's own
docs say, and fig has no segment that names a later one. So a set or a removal
changes the first declaration, and the one that applies stays as it was.

## Repro

On `main` (`41cd12c`):

```rust
let set = fig_css::with_declaration("fill: red; fill: blue", "fill", Some("green")).unwrap();
assert_eq!(set, "fill: green; fill: blue");
assert_eq!(fig_css::declaration(&set, "fill").as_deref(), Some("blue")); // not "green"

let gone = fig_css::with_declaration("fill: red; fill: blue", "fill", None).unwrap();
assert_eq!(gone, "fill: blue");
assert_eq!(fig_css::declaration(&gone, "fill").as_deref(), Some("blue")); // not None
```

The repeats don't have to be spelled the same way. `FILL: red; fill: blue` set
to `green` gives `FILL: green; fill: blue`, because `with_declaration` takes
the first spelling that `same_property` matches.

## What it costs

thorn and leaf call `with_declaration` to make a gesture's edit to a `style`
attribute. On a list that repeats the property, the gesture leaves the file
looking the same and reports no error. A removal is worse: it deletes a
declaration the reader never saw take effect and leaves the one that did. Few
writers repeat a property, but a hand-edited or concatenated `style` can.

## Candidates

- **Edit the last one.** Find the applied declaration with `declarations`,
  and splice its value, or the declaration and its `;`, by span instead of by
  path. Earlier repeats stay where they are, overridden as they already were.
  This needs the value's span, which `Declaration` does not carry yet. The
  node table has it.
- **Collapse the repeats.** Set the last and delete the earlier ones. The
  result is tidier, but it removes bytes the caller did not ask to change.

A removal has to take every repeat. Taking only the last brings the one before
it into effect.

## Done when

For any list that parses, `declaration(&with_declaration(list, p, Some(v))?, p)`
is `Some(v)` and `declaration(&with_declaration(list, p, None)?, p)` is `None`.
This holds when `p` repeats, with the same spelling or a different case. A test
in `src/lib.rs` holds both assertions.
