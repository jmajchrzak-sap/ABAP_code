# ABAP 7.4 / 7.5 New Syntax Reference (Old vs. New)

Applies to classic ECC/NetWeaver AS ABAP. Version tags:
- **[7.40]** available from NW 7.40 SP05 onward
- **[7.50]** available from NW 7.50 onward

---

## 1. Inline Declarations `[7.40]`

**Old:**
```abap
DATA: lv_matnr TYPE matnr,
      ls_mara  TYPE mara,
      lt_mara  TYPE TABLE OF mara.

SELECT SINGLE * FROM mara INTO ls_mara WHERE matnr = lv_matnr.
```

**New:**
```abap
SELECT SINGLE * FROM mara INTO @DATA(ls_mara) WHERE matnr = @lv_matnr.
```

Also works for field-symbols and method call results:
```abap
LOOP AT lt_mara ASSIGNING FIELD-SYMBOL(<mara>).
  ...
ENDLOOP.

DATA(lv_result) = go_calc->compute( 10 ).
```

Caveat: `DATA(...)` fixes the type at the point of declaration — you can't reuse the name with a different type in the same scope, and it's less obvious at a glance what type you're getting when reading code later (worth weighing for maintainability in shared code).

---

## 2. String Templates `|...|` `[7.40]`

**Old:**
```abap
DATA(lv_msg) = 'Material ' && lv_matnr && ' has ' && lv_qty && ' units'.
CONCATENATE 'Material' lv_matnr INTO lv_msg SEPARATED BY space.
```

**New:**
```abap
DATA(lv_msg) = |Material { lv_matnr } has { lv_qty } units|.

" with formatting options
DATA(lv_msg2) = |Amount: { lv_amount NUMBER = USER }|.
DATA(lv_date) = |Date: { sy-datum DATE = ISO }|.
DATA(lv_pad)  = |ID: { lv_id WIDTH = 10 ALIGN = LEFT PAD = '0' }|.
```

---

## 3. Table Expressions `[7.40]`

**Old:**
```abap
READ TABLE lt_mara INTO ls_mara WITH KEY matnr = lv_matnr.
IF sy-subrc = 0.
  " use ls_mara
ENDIF.

READ TABLE lt_mara INTO ls_mara INDEX 1.
```

**New:**
```abap
TRY.
    DATA(ls_mara) = lt_mara[ matnr = lv_matnr ].
  CATCH cx_sy_itab_line_not_found.
    " not found
ENDTRY.

" by index
ls_mara = lt_mara[ 1 ].

" combined with COND to avoid the TRY/CATCH for a default
DATA(ls_mara2) = VALUE #( lt_mara[ matnr = lv_matnr ] OPTIONAL ).
```

Note the important semantic shift: table expressions **raise an exception** (`CX_SY_ITAB_LINE_NOT_FOUND`) when nothing is found, instead of setting `sy-subrc`. Always wrap in `TRY...CATCH` or use `OPTIONAL`/`DEFAULT` inside `VALUE #( )`.

---

## 4. Constructor Operators

### `VALUE` — building structures/tables `[7.40]`

**Old:**
```abap
DATA: ls_mara TYPE mara.
ls_mara-matnr = lv_matnr.
ls_mara-mtart = 'FERT'.

DATA: lt_range TYPE RANGE OF matnr.
DATA: ls_range LIKE LINE OF lt_range.
ls_range-sign   = 'I'.
ls_range-option = 'EQ'.
ls_range-low    = lv_matnr.
APPEND ls_range TO lt_range.
```

**New:**
```abap
DATA(ls_mara) = VALUE mara( matnr = lv_matnr mtart = 'FERT' ).

DATA(lt_range) = VALUE rseloption( ( sign = 'I' option = 'EQ' low = lv_matnr ) ).

" table from a loop / mapping
DATA(lt_out) = VALUE tt_out( FOR ls_in IN lt_in
                              ( matnr = ls_in-matnr text = ls_in-maktx ) ).
```

### `NEW` — object instantiation `[7.40]`

**Old:**
```abap
DATA: lo_obj TYPE REF TO zcl_my_class.
CREATE OBJECT lo_obj
  EXPORTING
    iv_id = lv_id.
```

**New:**
```abap
DATA(lo_obj) = NEW zcl_my_class( iv_id = lv_id ).

" or, when passed directly into a method call:
go_processor->process( NEW zcl_my_class( iv_id = lv_id ) ).
```

### `CORRESPONDING` — structure/table mapping `[7.40]`

**Old:**
```abap
MOVE-CORRESPONDING ls_source TO ls_target.

CLEAR lt_target.
LOOP AT lt_source INTO ls_source.
  MOVE-CORRESPONDING ls_source TO ls_target.
  APPEND ls_target TO lt_target.
ENDLOOP.
```

**New:**
```abap
DATA(ls_target) = CORRESPONDING zzs_target( ls_source ).

DATA(lt_target) = CORRESPONDING tt_target( lt_source ).

" with explicit field mapping and excluding fields
DATA(lt_target2) = CORRESPONDING tt_target( lt_source
                     MAPPING matnr_out = matnr
                     EXCEPT  ernam ).
```

### `COND` / `SWITCH` — conditional expressions `[7.40]`

**Old:**
```abap
IF lv_status = 'A'.
  lv_text = 'Active'.
ELSEIF lv_status = 'I'.
  lv_text = 'Inactive'.
ELSE.
  lv_text = 'Unknown'.
ENDIF.

CASE lv_status.
  WHEN 'A'. lv_text = 'Active'.
  WHEN 'I'. lv_text = 'Inactive'.
  WHEN OTHERS. lv_text = 'Unknown'.
ENDCASE.
```

**New:**
```abap
DATA(lv_text) = COND string( WHEN lv_status = 'A' THEN 'Active'
                              WHEN lv_status = 'I' THEN 'Inactive'
                              ELSE 'Unknown' ).

DATA(lv_text2) = SWITCH string( lv_status
                                 WHEN 'A' THEN 'Active'
                                 WHEN 'I' THEN 'Inactive'
                                 ELSE 'Unknown' ).
```

### `REDUCE` — aggregation `[7.40]`

**Old:**
```abap
DATA: lv_total TYPE p DECIMALS 2.
LOOP AT lt_items INTO ls_item.
  lv_total = lv_total + ls_item-amount.
ENDLOOP.
```

**New:**
```abap
DATA(lv_total) = REDUCE decfloat34( INIT sum = 0
                                     FOR ls_item IN lt_items
                                     NEXT sum = sum + ls_item-amount ).
```

### `FILTER` — filtering tables `[7.40]`

**Old:**
```abap
CLEAR lt_result.
LOOP AT lt_items INTO ls_item WHERE status = 'A'.
  APPEND ls_item TO lt_result.
ENDLOOP.
```

**New:**
```abap
DATA(lt_result) = FILTER #( lt_items WHERE status = 'A' ).
```
Note: `FILTER` requires the WHERE field(s) to be part of a **sorted/hashed secondary or primary key** on the source table for anything beyond simple equality — this trips people up coming from `LOOP AT ... WHERE`.

### `CONV` — explicit type conversion `[7.40]`

**Old:**
```abap
DATA: lv_str TYPE string.
lv_str = lv_matnr.  " implicit conversion, sometimes ambiguous in expressions
```

**New:**
```abap
" mainly needed to fix a type inside a constructor expression / method call
go_obj->set_value( CONV string( lv_matnr ) ).
```

---

## 5. Functional (nested) method calls `[7.40]`

**Old:**
```abap
DATA: lv_result1 TYPE string,
      lv_result2 TYPE string.

lv_result1 = go_obj->get_name( ).
lv_result2 = go_other->format( lv_result1 ).
```

**New:**
```abap
DATA(lv_result2) = go_other->format( go_obj->get_name( ) ).
```
Methods can now be called directly as operands wherever a value is expected — no need to stage the result in a helper variable first. Only works for methods with a `RETURNING` parameter (still true in 7.4/7.5, not for `EXPORTING`).

---

## 6. `LOOP AT ... GROUP BY` `[7.40]`

**Old:**
```abap
SORT lt_items BY status.
LOOP AT lt_items INTO ls_item.
  AT NEW status.
    " group start logic — awkward, control-break syntax
  ENDAT.
  ...
  AT END OF status.
    " group end logic
  ENDAT.
ENDLOOP.
```

**New:**
```abap
LOOP AT lt_items INTO ls_item
     GROUP BY ls_item-status
     INTO DATA(ls_group)
     ASCENDING.

  DATA(lv_count) = 0.
  LOOP AT GROUP ls_group INTO DATA(ls_member).
    lv_count += 1.
  ENDLOOP.

  WRITE: / ls_group, lv_count.
ENDLOOP.
```

---

## 7. `INSERT`/`MODIFY` with `VALUE` inline `[7.40]`

**Old:**
```abap
DATA: ls_line TYPE ty_line.
ls_line-id   = 1.
ls_line-text = 'foo'.
INSERT ls_line INTO TABLE lt_tab.
```

**New:**
```abap
INSERT VALUE #( id = 1 text = 'foo' ) INTO TABLE lt_tab.
```

---

## 8. Boolean expressions & `xsdbool` `[7.40]`

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

" or, in newer code, boolean literal expressions directly:
DATA(lv_flag2) = COND abap_bool( WHEN lv_qty > 0 THEN abap_true ELSE abap_false ).
```

---

## 9. `RAISE EXCEPTION NEW` `[7.40]`

**Old:**
```abap
DATA: lx_error TYPE REF TO zcx_my_error.
CREATE OBJECT lx_error
  EXPORTING
    textid = zcx_my_error=>not_found.
RAISE EXCEPTION lx_error.
```

**New:**
```abap
RAISE EXCEPTION NEW zcx_my_error( textid = zcx_my_error=>not_found ).
```

---

## 10. `line_exists( )` / `line_index( )` `[7.40]`

**Old:**
```abap
READ TABLE lt_mara TRANSPORTING NO FIELDS WITH KEY matnr = lv_matnr.
IF sy-subrc = 0.
  " exists
ENDIF.
```

**New:**
```abap
IF line_exists( lt_mara[ matnr = lv_matnr ] ).
  " exists
ENDIF.

DATA(lv_idx) = line_index( lt_mara[ matnr = lv_matnr ] ).
```

---

## 11. `FOR` iteration expressions (comprehensions) `[7.40]`

**Old:**
```abap
DATA: lt_matnr TYPE TABLE OF matnr.
LOOP AT lt_mara INTO ls_mara.
  APPEND ls_mara-matnr TO lt_matnr.
ENDLOOP.
```

**New:**
```abap
DATA(lt_matnr) = VALUE tt_matnr( FOR ls_mara IN lt_mara ( ls_mara-matnr ) ).

" with a WHERE-like filter condition
DATA(lt_active) = VALUE tt_mara( FOR ls_mara IN lt_mara WHERE ( mtart = 'FERT' )
                                  ( ls_mara ) ).
```

---

## 12. Meshes `[7.40]` (niche, but worth knowing exists)

Used for navigating associated internal tables like an in-memory data model with `-\_` navigation syntax. Rare in day-to-day ECC dev; mentioning for completeness — ask if you want a worked example, since it's a bigger topic (`MESH-TYPE`, `\_association`).

---

## 13. `[7.50]`-specific additions

These need NW 7.50, so double-check your system release before using:

### Table expressions and constructor expressions directly in more operand positions

**New in 7.50:**
```abap
" constructor expressions now allowed in more places, e.g. directly in LOOP AT
LOOP AT VALUE tt_mara( FOR ls IN lt_mara WHERE ( mtart = 'FERT' ) ( ls ) )
     INTO DATA(ls_fert).
  ...
ENDLOOP.
```

### `NEW` for elementary types / `CAST`

**Old:**
```abap
DATA: lo_ref TYPE REF TO data.
CREATE DATA lo_ref TYPE i.
```

**New [7.50]:**
```abap
DATA(lo_ref) = NEW i( 42 ).
```

**Old (downcast):**
```abap
DATA: lo_child TYPE REF TO zcl_child.
lo_child ?= lo_parent.
```

**New [7.50]:**
```abap
DATA(lo_child) = CAST zcl_child( lo_parent ).
```

### Loop with a direct arithmetic expression / `REDUCE` refinements, and unary/binary calculation expressions in more contexts (e.g. directly on DB-selected inline data) become smoother — mostly ergonomic, no dramatically new keyword beyond what's above.

### `COND`/`SWITCH` allowed to throw exceptions directly `[7.50]`
```abap
DATA(lv_result) = COND #( WHEN lv_input < 0
                           THEN THROW zcx_invalid_input( )
                           ELSE lv_input ).
```

---

## Practical notes for ECC/NetWeaver classic dev

- Check your kernel/component release with `SM51`/`SPAM` info or ask Basis before assuming 7.50 features are available — many ECC 6.0 EHP7/EHP8 systems are on 7.40 or 7.31, where none of this works and you're stuck with classic syntax.
- `DATA(...)` inline declarations inside `LOOP` are scoped to the loop in older 7.40 SPs in a way that changed slightly across support packages — test if you see odd behavior reusing the name after the loop.
- Table expressions (`itab[ ... ]`) raising exceptions instead of setting `sy-subrc` is the single biggest gotcha for ABAP developers moving from classic to new syntax — code review carefully for missing `TRY`/`CATCH` or `OPTIONAL`/`DEFAULT`.
- `FILTER` needs a matching secondary/primary table key for the WHERE condition — plain unsorted internal tables will raise a syntax error if the key doesn't fit, unlike the flexible `LOOP AT ... WHERE`.
