---
title: A comment holding an opener cuts a value short
description: A comment after a value or a selector that has `/*` inside it ends the value at that inner `/*`, and an edit to the value deletes the comment's head
status: open
created: 2026-10-22
updated: 2026-10-22
part_of: "[Tasks](tasks.md)"
---

# A comment holding an opener cuts a value short

CSS comments do not nest: `/* was /* blue */` is one comment, opened at the
first `/*` and closed at the `*/`. `Parser::trim_end`, which takes a remark
off the end of a name, a value or a selector, finds the comment's start by
looking backwards with `rfind("/*")`, so it finds the inner `/*` and stops
there. Whatever the comment says before that is read as part of the value.

## Repro

```rust
let list = "fill: red /* was /* blue */";
assert_eq!(fig_css::declaration(list, "fill").as_deref(), Some("red"));
// fails: Some("red /* was")

assert_eq!(
    fig_css::with_declaration(list, "fill", Some("green")).unwrap(),
    "fill: green /* was /* blue */",
);
// fails: "fill: green /* blue */" — the splice replaced the value's span,
// which ran into the comment, so ` was` is gone

let css = "a /* x /* y */ { fill: red }";
let rules = fig_css::stylesheet(css).unwrap();
assert_eq!(&css[rules[0].prelude.clone()], "a");
// fails: "a /* x"
```

Checked against `main` at `d240088`.

## What it costs

A comment that quotes or comments out CSS is where this shows up, for
example `stroke: #000 /* stroke: #333; /* old */`. The value a caller reads
carries part of the comment, and an edit to that declaration or selector
destroys the comment's head. The crate promises to keep every byte it was
not asked to change.

## The change

Find the end of a name or a value by scanning forwards from its start, as
`trivia` and `scan` already do, rather than backwards from its end: step
over strings and over each comment from its own `/*` to its `*/`, and keep
the end of the last byte that is neither whitespace nor inside a comment.

## Done when

The three assertions above hold as tests in `src/lib.rs`, and
`declaration`, `with_declaration` and `stylesheet` keep their current
answers for a comment with no `/*` inside it.
