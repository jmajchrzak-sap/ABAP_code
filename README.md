# ABAP 7.4 / 7.5 Modern Syntax Guide

A practical, old-vs-new reference for the modern ABAP expression syntax introduced in NetWeaver 7.40/7.50 — written for developers coming from classic ECC/NetWeaver ABAP.

> **Target releases:** NW 7.40 SP05+ for `[7.40]`-tagged features, NW 7.50+ for `[7.50]`-tagged features. Check your system's kernel/SP level before assuming a feature is available — many ECC 6.0 systems still run 7.31/7.40.

## Contents

| Document | Covers |
|---|---|
| [`ABAP_7.5_New_Syntax_Reference.md`](./ABAP_7.5_New_Syntax_Reference.md) | Inline declarations, string templates, table expressions, constructor operators (`VALUE`, `NEW`, `CORRESPONDING`, `COND`, `SWITCH`, `REDUCE`, `FILTER`, `CONV`), functional method calls, `LOOP AT ... GROUP BY`, inline `INSERT`/`MODIFY`, `xsdbool`, `RAISE EXCEPTION NEW`, `line_exists`/`line_index`, `FOR` comprehensions, and 7.50-only additions (`CAST`, `NEW` for elementary types, exceptions inside `COND`) |
| [`ABAP_Built-in_Functions_Reference.md`](./ABAP_Built-in_Functions_Reference.md) | Built-in calculation functions (`ceil`, `floor`, `trunc`, `round`, `sign`, `abs`, `frac`, `nmax`, `nmin`, `ipow`), string functions (`shift_left`/`shift_right`, `find*`, `replace`, `substring*`, `condense`, `repeat`, `segment`, `escape`, `concat_lines_of`), and table functions (`lines`, `line_exists`, `line_index`) |

Both documents follow the same format throughout: an **Old** code block (classic ABAP), followed by a **New** code block (7.40/7.50 syntax), so you can diff the two approaches directly.

## Why this exists

Classic ABAP syntax (`READ TABLE ... WITH KEY`, `MOVE-CORRESPONDING`, `CONCATENATE`, control-break `AT NEW`/`AT END OF`, etc.) still works and is common in ECC/NetWeaver codebases. The 7.40/7.50 syntax additions are backward-compatible extensions, not replacements — this guide is meant as a lookup table when reading or modernizing existing code, not a mandate to rewrite everything.

## Key gotchas (read before using in production code)

- **Table expressions raise exceptions.** `itab[ key = value ]` raises `CX_SY_ITAB_LINE_NOT_FOUND` instead of setting `sy-subrc`. Always wrap in `TRY...CATCH` or use `VALUE #( itab[ ... ] OPTIONAL )`.
- **`FILTER` needs a matching table key.** The WHERE condition fields must be part of a primary or secondary table key, unlike the more flexible `LOOP AT ... WHERE`.
- **Offsets aren't consistently 0- or 1-based.** String function offsets (`find`, `substring`) are 0-based; `line_index` on an internal table returns a normal 1-based index, same as `sy-tabix`.
- **`DATA(...)` fixes the type at first declaration** — it can't be redeclared with a different type in the same scope, which matters for code review and readability in long procedures.

## Suggested repo layout

```
.
├── README.md
├── ABAP_7.5_New_Syntax_Reference.md
└── ABAP_Built-in_Functions_Reference.md
```

## License

Add a license of your choice (e.g. MIT) if you intend to make this repo public.
