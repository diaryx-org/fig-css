---
title: A nested rule whose selector has a colon is read as a declaration
description: Inside a declaration block, `&:hover { … }` or `a:hover { … }` is read as a property `&` or `a` whose value is `hover { … }`, so the nested rule is not in the tree and an edit to it is `NotFound`
status: open
created: 2026-10-29
updated: 2026-10-29
part_of: "[Tasks](tasks.md)"
---

# A nested rule whose selector has a colon is read as a declaration

The crate's docs say a declaration block may nest a rule, as CSS nesting
does. `Parser::declarations` tells a nested rule from a declaration by the
first of `:`, `;` or `{` it meets at depth zero: a `{` first is a rule, a
`:` first is a declaration. A selector can hold a colon before its `{`,
and in nested CSS it usually does: `&:hover`, `a:focus-visible`,
`&::before`, `:is(a, b)`. Each of those reaches the `:` first, so the
selector's head is read as the property and the rest, braces included, as
its value. Nothing is refused, so nothing says so.

## Repro

On `main` (`2fa0d71`):

```rust
use fig_css::Body;

let rules = fig_css::stylesheet("a { color: red; &:hover { color: blue } }\n").unwrap();
let Body::Declarations { declarations, rules: nested, .. } = &rules[0].body else { panic!() };
assert_eq!(nested.len(), 0); // not 1
assert_eq!(declarations[1].property, "&");
assert_eq!(declarations[1].value, "hover { color: blue }");

let mut ed = fig::Editor::open(
    b"a { color: red; &:hover { color: blue } }\n",
    fig_css::formats().unwrap().stylesheet,
).unwrap();
let path = [fig::Segment::Key("a"), fig::Segment::Key("&:hover"), fig::Segment::Key("color")];
assert!(ed.set_value(&path, "green").is_err()); // NotFound
```

`span:hover { color: blue }` in the same place reads as `span` over
`hover { color: blue }`. `& span { color: blue }`, with no colon, is read
as a nested rule, as it should be.

## What it costs

A caller that walks `stylesheet`'s rules misses every nested rule with a
pseudo-class or pseudo-element, and `Rule::declared` can answer for a
property named `&`. An edit to a declaration inside one is refused, since
the path the caller can see in the text is not in the tree. A stylesheet
thorn or leaf writes today has no nesting, so nothing ships broken. A
hand-written or imported sheet that nests is read wrong.

## Candidates

- **Read a block's contents as CSS Syntax 3 does.** Try a declaration
  first. It is one only if its name is an identifier (`&:hover`, `.a:hover`
  and `:hover` are not), and, unless it is a custom property (`--x`), if
  its value holds no `{…}` at depth zero before its `;`. Anything else is
  read again from its start as a nested rule. A custom property's value may
  hold braces, so `--x: { a: b }` stays a declaration.
- **Refuse what is ambiguous.** Keep the reading as it is, but refuse a
  declaration whose value holds a top-level `{`, outside a custom property,
  as an error at it. Nothing is read wrong, and nested rules with colons
  can't be opened at all.

## Done when

`a { color: red; &:hover { color: blue } }` and
`a { span:hover { color: blue } }` each read as one declaration block
holding the nested rule. A set to `a`/`&:hover`/`color` changes only that
value. `--x: { a: b }` is still a declaration.
