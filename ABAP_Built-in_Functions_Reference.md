# ABAP Built-in Functions Reference (Calculation / String / Table Operands)

These are the built-in functions usable directly as operands in expressions (inside `DATA( )`, `WRITE`, `IF`, method calls, `VALUE`, `COND`, etc.), introduced/extended with the 7.4/7.5 expression syntax. Version tags as before: **[7.40]** / **[7.50]**.

---

## 1. Numeric / calculation functions `[7.40]`

### `ceil` / `floor` / `trunc`

**Old:**
```abap
DATA: lv_result TYPE i.
CALL FUNCTION 'ROUND'  " or manual arithmetic
  EXPORTING
    input = lv_value
    ...
" or, common manual trunc:
lv_result = lv_value - ( lv_value MOD 1 ).
```

**New:**
```abap
DATA(lv_ceil)  = ceil( lv_value ).    " rounds up to nearest integer
DATA(lv_floor) = floor( lv_value ).   " rounds down to nearest integer
DATA(lv_trunc) = trunc( lv_value ).   " cuts off decimals (toward zero)
```
Example: `floor( '4.7' ) = 4`, `ceil( '4.2' ) = 5`, `trunc( '-4.7' ) = -4`, `floor( '-4.2' ) = -5`.

### `round`

**Old:**
```abap
CALL FUNCTION 'ROUND'
  EXPORTING
    input     = lv_value
    decimals  = 2
    sign      = '0'
  IMPORTING
    output    = lv_result.
```

**New:**
```abap
DATA(lv_result) = round( val = lv_value dec = 2 ).
```

### `sign`

**Old:**
```abap
IF lv_value > 0.
  lv_sign = 1.
ELSEIF lv_value < 0.
  lv_sign = -1.
ELSE.
  lv_sign = 0.
ENDIF.
```

**New:**
```abap
DATA(lv_sign) = sign( lv_value ).   " returns -1, 0, or 1
```

### `abs`

**Old:**
```abap
IF lv_value < 0.
  lv_result = lv_value * -1.
ELSE.
  lv_result = lv_value.
ENDIF.
```

**New:**
```abap
DATA(lv_result) = abs( lv_value ).
```

### `frac`

**New (no clean old equivalent — used to be manual subtraction):**
```abap
DATA(lv_decimal_part) = frac( lv_value ).   " e.g. frac( '4.75' ) = 0.75
```

### `nmax` / `nmin`

**Old:**
```abap
IF lv_a > lv_b.
  lv_result = lv_a.
ELSE.
  lv_result = lv_b.
ENDIF.
```

**New:**
```abap
DATA(lv_result) = nmax( val1 = lv_a val2 = lv_b ).
DATA(lv_min)    = nmin( val1 = lv_a val2 = lv_b ).
" both accept more than 2 values: nmax( val1 = a val2 = b val3 = c )
```

### `ipow` (integer power)

**Old:**
```abap
lv_result = lv_base ** lv_exp.   " ** operator already existed, still valid
```

**New (explicit, usable inline in expressions where `**` can't be used):**
```abap
DATA(lv_result) = ipow( base = lv_base exp = lv_exp ).
```

---

## 2. String functions `[7.40]`

### `to_upper` / `to_lower` / `to_mixed`

**Old:**
```abap
TRANSLATE lv_text TO UPPER CASE.
TRANSLATE lv_text TO LOWER CASE.
```

**New:**
```abap
DATA(lv_upper) = to_upper( lv_text ).
DATA(lv_lower) = to_lower( lv_text ).
DATA(lv_mixed) = to_mixed( val = lv_text ).
```

### `shift_left` / `shift_right`

**Old:**
```abap
SHIFT lv_text LEFT.
SHIFT lv_text LEFT BY 3 PLACES.
SHIFT lv_text RIGHT DELETING TRAILING space.
```

**New:**
```abap
DATA(lv_result1) = shift_left( val = lv_text places = 3 ).
DATA(lv_result2) = shift_left( val = lv_text sub = 'X' ).      " shift left past leading 'X' chars
DATA(lv_result3) = shift_right( val = lv_text circular = 2 ).  " circular shift
```

### `insert`

**Old:**
```abap
CONCATENATE lv_text(3) 'XYZ' lv_text+3 INTO lv_result.
```

**New:**
```abap
DATA(lv_result) = insert( val = lv_text sub = 'XYZ' off = 3 ).
```

### `replace`

**Old:**
```abap
REPLACE 'foo' IN lv_text WITH 'bar'.
REPLACE ALL OCCURRENCES OF 'foo' IN lv_text WITH 'bar'.
```

**New:**
```abap
DATA(lv_result) = replace( val = lv_text sub = 'foo' with = 'bar' occ = 0 ).
" occ = 0 -> all occurrences, occ = 1 -> first only (default), off/len also supported
DATA(lv_result2) = replace( val = lv_text regex = 'a+' with = 'X' ).
```

### `find` / `find_end` / `find_any_of` / `find_any_not_of`

**Old:**
```abap
FIND 'foo' IN lv_text MATCH OFFSET lv_off.
SEARCH lv_text FOR 'foo'.
```

**New:**
```abap
DATA(lv_off)  = find( val = lv_text sub = 'foo' ).          " -1 if not found
DATA(lv_off2) = find( val = lv_text regex = '\d+' ).
DATA(lv_off3) = find_end( val = lv_text sub = 'foo' ).       " last occurrence
DATA(lv_off4) = find_any_of( val = lv_text sub = 'xyz' ).    " first char matching any of x/y/z
```

### `count` / `count_any_of` / `count_any_not_of`

**Old:**
```abap
" typically a manual loop counting occurrences
```

**New:**
```abap
DATA(lv_cnt) = count( val = lv_text sub = 'a' ).
DATA(lv_cnt2) = count( val = lv_text regex = '[0-9]' ).
```

### `match`

**New (regex capture, no clean old equivalent besides FIND REGEX + SUBMATCHES):**
```abap
DATA(lv_matched) = match( val = lv_text regex = '\d{4}-\d{2}-\d{2}' ).
```

### `substring` / `substring_before` / `substring_after` / `substring_from` / `substring_to`

**Old:**
```abap
lv_result = lv_text+2(5).                     " offset/length syntax
FIND 'foo' IN lv_text MATCH OFFSET lv_off.
lv_before = lv_text(lv_off).
```

**New:**
```abap
DATA(lv_sub)    = substring( val = lv_text off = 2 len = 5 ).
DATA(lv_before) = substring_before( val = lv_text sub = 'foo' ).
DATA(lv_after)  = substring_after( val = lv_text sub = 'foo' ).
DATA(lv_from)   = substring_from( val = lv_text sub = 'foo' ).  " includes 'foo' onward
DATA(lv_to)     = substring_to( val = lv_text sub = 'foo' ).    " up to and including 'foo'
```

### `condense`

**Old:**
```abap
CONDENSE lv_text.
CONDENSE lv_text NO-GAPS.
```

**New:**
```abap
DATA(lv_result) = condense( val = lv_text ).
DATA(lv_result2) = condense( val = lv_text del = ` ` ).
```

### `repeat`

**New (no direct old equivalent besides a manual loop/DO):**
```abap
DATA(lv_result) = repeat( val = '-' occ = 10 ).   " '----------'
```

### `escape`

**New (used for embedding untrusted content, e.g. into HTML/URL/SQL contexts):**
```abap
DATA(lv_escaped) = escape( val = lv_text format = cl_abap_format=>e_html_text ).
```

### `concat_lines_of`

**Old:**
```abap
LOOP AT lt_lines INTO lv_line.
  CONCATENATE lv_result lv_line INTO lv_result.
ENDLOOP.
```

**New:**
```abap
DATA(lv_result) = concat_lines_of( table = lt_lines sep = cl_abap_char_utilities=>newline ).
```

### `segment`

**New (splits string by delimiter and returns nth segment):**
```abap
DATA(lv_part) = segment( val = lv_text sep = '/' index = 2 ).
```

---

## 3. Table functions `[7.40]`

### `lines`

**Old:**
```abap
DESCRIBE TABLE lt_tab LINES lv_count.
```

**New:**
```abap
DATA(lv_count) = lines( lt_tab ).
```

### `line_exists`

**Old:**
```abap
READ TABLE lt_tab TRANSPORTING NO FIELDS WITH KEY matnr = lv_matnr.
IF sy-subrc = 0.
  ...
ENDIF.
```

**New:**
```abap
IF line_exists( lt_tab[ matnr = lv_matnr ] ).
  ...
ENDIF.
```

### `line_index`

**Old:**
```abap
READ TABLE lt_tab TRANSPORTING NO FIELDS WITH KEY matnr = lv_matnr.
lv_idx = sy-tabix.
```

**New:**
```abap
DATA(lv_idx) = line_index( lt_tab[ matnr = lv_matnr ] ).   " 0 if not found
```

---

## 4. Type / reference functions `[7.40]`/`[7.50]`

### `xsdbool` `[7.40]`

**Old:**
```abap
IF lv_qty > 0.
  lv_flag = abap_true.
ELSE.
  lv_flag = abap_false.
ENDIF.
```

**New:**
```abap
DATA(lv_flag) = xsdbool( lv_qty > 0 ).
```

### `cast` (operator, not function) `[7.50]` — see previous reference doc for full example

### `ref #( )` — get a reference to a value `[7.40]`

**Old:**
```abap
DATA: lv_value TYPE i VALUE 42,
      lo_ref   TYPE REF TO i.
GET REFERENCE OF lv_value INTO lo_ref.
```

**New:**
```abap
DATA(lo_ref) = REF #( lv_value ).
```

---

## Quick lookup table

| Function | Category | Old-world equivalent |
|---|---|---|
| `ceil`, `floor`, `trunc` | numeric | manual arithmetic / `ROUND` FM |
| `round` | numeric | `ROUND` FM |
| `sign` | numeric | `IF`/`ELSEIF` chain |
| `abs` | numeric | `IF lv < 0` negate |
| `frac` | numeric | manual subtraction |
| `nmax`, `nmin` | numeric | `IF a > b` |
| `ipow` | numeric | `**` operator |
| `to_upper`, `to_lower`, `to_mixed` | string | `TRANSLATE ... TO UPPER/LOWER CASE` |
| `shift_left`, `shift_right` | string | `SHIFT` |
| `insert` | string | `CONCATENATE` with offsets |
| `replace` | string | `REPLACE` |
| `find`, `find_end`, `find_any_of` | string | `FIND`/`SEARCH` |
| `count`, `count_any_of` | string | manual loop |
| `match` | string | `FIND ... REGEX ... SUBMATCHES` |
| `substring*` | string | offset/length syntax `text+n(m)` |
| `condense` | string | `CONDENSE` |
| `repeat` | string | manual `DO` loop |
| `escape` | string | manual encoding logic |
| `concat_lines_of` | string | `LOOP` + `CONCATENATE` |
| `segment` | string | `SPLIT` + index |
| `lines` | table | `DESCRIBE TABLE ... LINES` |
| `line_exists` | table | `READ TABLE ... TRANSPORTING NO FIELDS` |
| `line_index` | table | `READ TABLE` + `sy-tabix` |
| `xsdbool` | boolean | `IF`/`ELSE` setting `abap_true`/`abap_false` |
| `REF #( )` | reference | `GET REFERENCE OF` |

---

## Notes for classic ECC/NetWeaver

- All of these are string/table/numeric *functions*, distinct from the constructor *operators* (`VALUE`, `NEW`, `COND`, etc.) covered in the syntax reference — functions take value operands and return a value; operators build/cast types.
- `find`/`count`/`match`/`replace` with `regex =` use POSIX-like ABAP regex syntax — same engine as `FIND ... REGEX`, so existing regex knowledge carries over directly.
- Offsets and indices from these functions (`find`, `line_index`) are **0-based for string offsets** but table `line_index` returns a normal 1-based table index — easy to mix up.
- Since these are usable inline anywhere an operand is expected, you'll commonly see them nested: `DATA(lv_x) = to_upper( substring( val = lv_text len = 5 ) ).`
