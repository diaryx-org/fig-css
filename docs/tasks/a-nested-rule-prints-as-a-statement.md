---
title: A rule holding a nested rule prints its declarations as statements
description: '`Css::print` decides a block holds rules when any member is a block, so a declaration block with a nested rule prints `color: red` as `color red;`, which fig-css then refuses to parse'
status: open
created: 2026-11-08
updated: 2026-11-08
part_of: "[Tasks](tasks.md)"
---

# A rule holding a nested rule prints its declarations as statements

The parser decides what a block holds by what opens it. An at-rule
`holds_rule_list` names holds rules, and every other block holds
declarations, with a nested rule allowed among them. `rules_under` asks the
same question. The printer does not ask it. `print_rules` calls `holds_rules`,
which answers yes for a block where any member's value is a mapping. A
declaration block that nests a rule, as CSS nesting allows and the parser
reads, is then printed as a block of rules, and each of its plain
declarations goes through the at-rule statement branch: the property, a
space, the value and a `;`, with no colon.

## Repro

On `main` (`601c5c2`):

```rust
let f = fig_css::formats().unwrap();
let src = "a { color: red; &:hover { color: blue } }\n";
let doc = fig::Document::parse(src.as_bytes(), f.stylesheet).unwrap();
let out = doc.serialize(f.stylesheet).unwrap();
assert_eq!(out, "a {\n  color red;\n  &:hover { color: blue }\n}\n");
// fails: expected `:` after a property, at byte 6
fig::Document::parse(out.as_bytes(), f.stylesheet).unwrap();
```

`Css.print(STYLESHEET, …)` on the table `Css.parse` makes gives the same
bytes. A sheet with no nested rule prints and reads back. So does an
`@media` block of rules.

## What it costs

Anything fig prints through the language, rather than splices, writes CSS
that changes meaning and does not parse: `Document::serialize` to `css`, a
conversion into it, and `fig` on the command line through
`examples/css_helper.rs`. The editor's splices do not print a whole block, so
thorn's and leaf's edits are not affected today. fig's load-time harness
round-trips only the two `samples`, and neither nests a rule.

## Done when

The printer decides a block's kind the way the parser does, by the rule's
prelude through `holds_rule_list` and not by its members. The repro prints
`a { color: red; &:hover { color: blue } }\n` or another spelling that reads
back to the same tree. A test holds the round trip for a nested rule, for an
`@media` block of rules, and for an empty block of each kind.
