# fig-css

The CSS [thorn](https://github.com/diaryx-org/thorn) and
[leaf](https://github.com/diaryx-org/leaf) edit — a stylesheet and a `style`
attribute — as a [fig](https://github.com/diaryx-org/fig) language, so that
every edit is a splice that keeps the bytes it was not asked to change: the
comments, the spacing, a property another app wrote.

```rust
let style = "color: #333; /* theirs */ text-align: center";
assert_eq!(fig_css::declaration(style, "text-align").as_deref(), Some("center"));
assert_eq!(
    fig_css::with_declaration(style, "text-align", Some("right"))?,
    "color: #333; /* theirs */ text-align: right"
);
```

It is not a general CSS parser and is not meant to become one. It reads what
its two callers need, as written, and decodes no value.

| function | what it does |
|---|---|
| `declaration(list, property)` | the value a `style` gives a property, as written; the last where it repeats |
| `declarations(list)` | every declaration of a `style`, in order, with its span |
| `with_declaration(list, property, value)` | the `style` with one property set, changed or (for `None`) taken out, and every other byte kept |
| `stylesheet(css)` | a stylesheet's rules, each with the spans of its parts, so a caller splices where the parser says |
| `selectors(list)` | a selector list split at its top-level commas |
| `formats()` | the two registered formats, `css` and `css-declarations`, for anything else fig does with them |

`examples/css_helper.rs` serves the language to the `fig` command line over
stdio, so `fig` reads and edits a stylesheet through the same parser.

## License

MIT or Apache-2.0, at your option.
