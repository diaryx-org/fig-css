---
title: An at-rule holding declarations is known only in lower case
description: '`@FONT-FACE`, `@Page` and the other declaration at-rules spelled in any case but lower are read as holding rules, so their declarations are refused as selectors and the whole stylesheet fails to parse'
status: open
created: 2026-10-30
updated: 2026-10-30
part_of: "[Tasks](tasks.md)"
---

# An at-rule holding declarations is known only in lower case

CSS compares an at-rule's name without regard to ASCII case (CSS 2.1
§4.1.3: everything in a stylesheet is case-insensitive in the ASCII range
except what CSS does not control), so `@Page` is `@page` and `@FONT-FACE`
is `@font-face`. `holds_rule_list` decides whether an at-rule's block
holds rules or declarations by looking its name up in
`DECLARATION_AT_RULES` with `contains`, which compares bytes. A name in
any other case is not found. The block is then read as a list of rules,
its first declaration has no `{`, and the parser refuses the stylesheet.

`same_property` already compares property names this way. At-rule names
are the case it misses.

## Repro

On `main` (`87650a1`):

```rust
assert!(fig_css::stylesheet("@font-face { font-family: X; src: url(a.woff) }\n").is_ok());

let err = fig_css::stylesheet("@FONT-FACE { font-family: X; src: url(a.woff) }\n").unwrap_err();
assert_eq!(err.message, "expected `{` after a selector"); // at byte 13

assert!(fig_css::stylesheet("@Page { margin: 1in }\n").is_err());
assert!(fig_css::stylesheet("@Property --x { syntax: '<length>'; inherits: false }\n").is_err());

let sheet = fig_css::formats().unwrap().stylesheet;
assert!(fig::Editor::open(b"@Page { margin: 1in }\n", sheet).is_err());
```

An at-rule that holds rules (`@MEDIA print { a { color: red } }`) reads
correctly, because a name not in the list is taken to hold rules either way.

## What it costs

A stylesheet with one such at-rule does not parse at all, so `stylesheet`
returns no rules and fig's editor will not open it. Nothing thorn or leaf
writes spells an at-rule in upper case, so nothing ships broken. A
hand-written or imported sheet that does is refused, though a browser
reads it.

## Done when

`holds_rule_list` matches an at-rule's name against
`DECLARATION_AT_RULES` without regard to ASCII case, the repro's sheets
parse with their declarations read as declarations, and a test holds it.
The prelude stays as written: only the comparison changes.
