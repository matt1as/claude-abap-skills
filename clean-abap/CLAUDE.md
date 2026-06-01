# Clean ABAP — always-on rules

Universal rules for any modern ABAP work (BTP ABAP Environment or S/4HANA on-prem in ABAP Cloud development model). Derived from the [SAP Clean ABAP styleguide](https://github.com/SAP/styleguides/blob/main/clean-abap/CleanABAP.md), distilled into AI-enforceable form.

These rules apply **before** the ABAP Cloud / RAP overlay. The overlay refines and extends them — it does not relax them.

When generating or reviewing ABAP, apply every rule below. When two rules appear to conflict, prefer the one with the stronger consequence stated in the **Why** line.

---

## RULE: use-problem-domain-names
**Why**: Technical or abbreviated names (`lv_tab1`, `l_str`, `it_tab`) force every reader to reverse-engineer intent from usage, slowing reviews and hiding bugs in name/value mismatches.
**Do**:
```abap
DATA(open_sales_orders)        = sales_order_reader->read_open( customer_id ).
DATA(max_retry_count)          = 5.
CONSTANTS status_blocked       TYPE i_salesorder-overallstatus VALUE 'B'.
```
**Avoid**:
```abap
DATA: lt_tab1 TYPE STANDARD TABLE OF zsorder,
      lv_i    TYPE i VALUE 5,
      lc_b    TYPE c LENGTH 1 VALUE 'B'.
```
**ATC**: code pal for ABAP — *Naming Conventions* / *Prefix Notation*

---

## RULE: no-magic-numbers-or-literals
**Why**: A bare `'B'` or `42` in a condition is unsearchable, unexplainable, and impossible to change safely — the next reader has no way to know what else depends on the same value.
**Do**:
```abap
CONSTANTS status_blocked TYPE i_salesorder-overallstatus VALUE 'B'.

IF order-overallstatus = status_blocked.
  RAISE EXCEPTION NEW zcx_order_blocked( order_id = order-salesorder ).
ENDIF.
```
**Avoid**:
```abap
IF order-overallstatus = 'B'.
  RAISE EXCEPTION NEW zcx_order_blocked( order_id = order-salesorder ).
ENDIF.
```
**ATC**: code pal for ABAP — *Magic Number*

---

## RULE: prefer-inline-declarations
**Why**: Up-front `DATA:` blocks separate a variable's declaration from its first use, defeating type inference and making refactoring noisier. Inline declarations make scope obvious and remove unused-variable risk.
**Do**:
```abap
DATA(order) = sales_order_reader->read_single( order_id ).
DATA(line_count) = lines( order-items ).
```
**Avoid**:
```abap
DATA: order      TYPE zsales_order,
      line_count TYPE i.

order      = sales_order_reader->read_single( order_id ).
line_count = lines( order-items ).
```
**ATC**: code pal for ABAP — *Chained Statement Usage*

---

## RULE: no-default-key-on-internal-tables
**Why**: `DEFAULT KEY` builds an implicit key from all non-numeric components — slow, surprising, and silently changing behaviour when fields are added. Always state the key (or `EMPTY KEY`) explicitly.
**Do**:
```abap
DATA orders TYPE SORTED TABLE OF zsales_order
            WITH UNIQUE KEY salesorder.

DATA error_messages TYPE STANDARD TABLE OF string
                    WITH EMPTY KEY.
```
**Avoid**:
```abap
DATA orders          TYPE STANDARD TABLE OF zsales_order WITH DEFAULT KEY.
DATA error_messages  TYPE STANDARD TABLE OF string       WITH DEFAULT KEY.
```
**ATC**: SAP standard ATC — *Internal Table with DEFAULT KEY*

---

## RULE: use-string-templates-not-concatenate
**Why**: `CONCATENATE` with explicit separators is verbose, error-prone, and obscures the intended message structure. String templates show the literal text with variable interpolation in place.
**Do**:
```abap
DATA(message) = |Order { order_id } rejected: status is { status } (expected { expected_status })|.
```
**Avoid**:
```abap
DATA message TYPE string.
CONCATENATE 'Order' order_id 'rejected: status is' status
            'expected' expected_status INTO message SEPARATED BY space.
```
**ATC**: code pal for ABAP — *Prefer String Template to & Operator*

---

## RULE: prefer-is-not-initial-over-negation
**Why**: `NOT ... IS INITIAL` forces a mental double-negation. `IS NOT INITIAL` reads in the same direction as the data flow, with no precedence ambiguity.
**Do**:
```abap
IF order-customer_id IS NOT INITIAL.
  process_customer_order( order ).
ENDIF.

IF line_exists( orders[ salesorder = order_id ] ).
  ...
ENDIF.
```
**Avoid**:
```abap
IF NOT order-customer_id IS INITIAL.
  process_customer_order( order ).
ENDIF.

IF NOT line_exists( orders[ salesorder = order_id ] ).
  RETURN.
ENDIF.
```
**ATC**: code pal for ABAP — *Equals Sign Chaining* / *IS NOT* preference

---

## RULE: prefer-case-over-long-if-elseif
**Why**: A chain of `IF ... ELSEIF ... ELSEIF` on the same variable obscures that the branches are mutually exclusive on a single discriminator, and grows worse with every new case. `CASE` makes exclusivity explicit and lets `WHEN OTHERS` enforce completeness.
**Do**:
```abap
CASE order-overallstatus.
  WHEN status_open.
    process_open_order( order ).
  WHEN status_blocked.
    raise_blocked( order ).
  WHEN status_closed.
    archive_order( order ).
  WHEN OTHERS.
    RAISE EXCEPTION NEW zcx_unknown_status( status = order-overallstatus ).
ENDCASE.
```
**Avoid**:
```abap
IF order-overallstatus = 'O'.
  process_open_order( order ).
ELSEIF order-overallstatus = 'B'.
  raise_blocked( order ).
ELSEIF order-overallstatus = 'C'.
  archive_order( order ).
ENDIF.
```
**ATC**: code pal for ABAP — *Nesting Depth* / *Cyclomatic Complexity*

---

## RULE: use-table-expressions-not-read-table-plus-sy-subrc
**Why**: `READ TABLE ... WITH KEY` followed by `IF sy-subrc = 0` is two statements where one would do, leaks `sy-subrc` into surrounding logic, and is easy to forget. Table expressions express intent directly; `line_exists` and `line_index` cover the presence checks.
**Do**:
```abap
IF line_exists( orders[ salesorder = order_id ] ).
  DATA(order) = orders[ salesorder = order_id ].
  process( order ).
ENDIF.

TRY.
    DATA(order) = orders[ salesorder = order_id ].
  CATCH cx_sy_itab_line_not_found.
    RAISE EXCEPTION NEW zcx_order_not_found( order_id = order_id ).
ENDTRY.
```
**Avoid**:
```abap
DATA order TYPE zsales_order.
READ TABLE orders INTO order WITH KEY salesorder = order_id.
IF sy-subrc = 0.
  process( order ).
ENDIF.
```
**ATC**: SAP standard ATC — *Avoidable READ TABLE*

---

## RULE: methods-do-one-thing-and-stay-small
**Why**: A method that does more than one thing cannot be named accurately, cannot be tested in isolation, and grows beyond what a reviewer can hold in their head. Single-purpose methods are the smallest unit of reuse.
**Do**:
```abap
METHOD release_order.
  validate_order( order ).
  reserve_stock( order ).
  send_release_event( order ).
ENDMETHOD.

METHOD validate_order.
  IF order-quantity <= 0.
    RAISE EXCEPTION NEW zcx_invalid_quantity( order_id = order-salesorder ).
  ENDIF.
ENDMETHOD.
```
**Avoid**:
```abap
METHOD release_order.
  IF order-quantity <= 0.
    RAISE EXCEPTION NEW zcx_invalid_quantity( order_id = order-salesorder ).
  ENDIF.
  SELECT SINGLE * FROM zstock WHERE matnr = @order-material INTO @DATA(stock).
  IF stock-quantity < order-quantity. RAISE EXCEPTION NEW zcx_out_of_stock( ). ENDIF.
  UPDATE zstock SET quantity = quantity - @order-quantity WHERE matnr = @order-material.
  cl_bgmc_process_factory=>get_default_factory( )->create( ... ).
ENDMETHOD.
```
**ATC**: code pal for ABAP — *Method Length*

---

## RULE: at-most-three-importing-parameters
**Why**: A method with more than three importing parameters almost always has more than one responsibility. Long parameter lists are easy to call wrong (especially in ABAP where optional parameters can be silently omitted) and resist refactoring.
**Do**:
```abap
METHODS schedule_delivery
  IMPORTING order_id          TYPE zsales_order_id
            requested_date    TYPE dats
            carrier_selection TYPE zcarrier_selection.
```
**Avoid**:
```abap
METHODS schedule_delivery
  IMPORTING order_id            TYPE zsales_order_id
            requested_date      TYPE dats
            carrier_id          TYPE zcarrier_id
            carrier_service     TYPE zcarrier_service
            allow_partial       TYPE abap_bool OPTIONAL
            allow_express       TYPE abap_bool OPTIONAL
            cost_limit          TYPE p LENGTH 13 DECIMALS 2 OPTIONAL
            requested_by_user   TYPE syuname OPTIONAL.
```
**ATC**: code pal for ABAP — *Number of Method Parameters*

---

## RULE: prefer-returning-over-exporting
**Why**: `RETURNING` enables functional call style (`DATA(x) = obj->calc( ... )`), removes a whole class of "I forgot to read the EXPORTING parameter" bugs, and signals that the method has one output instead of several.
**Do**:
```abap
METHODS calculate_total
  IMPORTING order_id     TYPE zsales_order_id
  RETURNING VALUE(total) TYPE zsales_total
  RAISING   zcx_order_not_found.

DATA(total) = calculator->calculate_total( order_id ).
```
**Avoid**:
```abap
METHODS calculate_total
  IMPORTING order_id TYPE zsales_order_id
  EXPORTING total    TYPE zsales_total
            currency TYPE waers
            line_cnt TYPE i.

DATA total    TYPE zsales_total.
DATA currency TYPE waers.
DATA line_cnt TYPE i.
calculator->calculate_total(
  EXPORTING order_id = order_id
  IMPORTING total    = total
            currency = currency
            line_cnt = line_cnt ).
```
**ATC**: code pal for ABAP — *Use RETURNING Instead of EXPORTING*

---

## RULE: class-based-exceptions-not-sy-subrc
**Why**: Class-based exceptions carry context (custom attributes, `previous`, message), force callers to handle them, and survive refactoring. Method-level `sy-subrc` is invisible at the call site, silently zero when not set, and quietly forgotten.
**Do**:
```abap
METHODS read_single
  IMPORTING order_id      TYPE zsales_order_id
  RETURNING VALUE(result) TYPE zsales_order
  RAISING   zcx_order_not_found.

TRY.
    DATA(order) = sales_order_reader->read_single( order_id ).
  CATCH zcx_order_not_found INTO DATA(error).
    log->error( error->get_text( ) ).
    RAISE EXCEPTION NEW zcx_order_processing_failed( previous = error ).
ENDTRY.
```
**Avoid**:
```abap
sales_order_reader->read_single(
  EXPORTING order_id = order_id
  IMPORTING result   = DATA(order)
            ev_subrc = DATA(subrc) ).
IF subrc <> 0. RETURN. ENDIF.
```
**ATC**: SAP standard ATC — *Check on SY-SUBRC after Method Call*

---

## RULE: catch-specific-exceptions-not-cx-root
**Why**: `CATCH cx_root` swallows everything including programming errors and resource exhaustion, hiding the bug you would otherwise have caught in test. A specific catch documents which failures the code is actually prepared to handle.
**Do**:
```abap
TRY.
    DATA(order) = sales_order_reader->read_single( order_id ).
  CATCH zcx_order_not_found INTO DATA(not_found).
    log_warning( not_found->get_text( ) ).
  CATCH zcx_order_locked INTO DATA(locked).
    retry_later( locked->lock_owner ).
ENDTRY.
```
**Avoid**:
```abap
TRY.
    DATA(order) = sales_order_reader->read_single( order_id ).
  CATCH cx_root.
    " what just happened? we will never know
ENDTRY.
```
**ATC**: SAP standard ATC — *CATCH for too generic exception class CX_ROOT*

---

## RULE: final-classes-and-private-members-by-default
**Why**: Non-`FINAL` classes invite subclassing that was never designed for; public attributes invite coupling that survives forever. `FINAL` and `PRIVATE` by default mean every relaxation is an explicit design decision instead of an accident.
**Do**:
```abap
CLASS zcl_sales_order_processor DEFINITION
  PUBLIC FINAL
  CREATE PRIVATE.

  PUBLIC SECTION.
    INTERFACES zif_sales_order_processor.
    CLASS-METHODS create
      RETURNING VALUE(result) TYPE REF TO zif_sales_order_processor.

  PRIVATE SECTION.
    DATA reader TYPE REF TO zif_sales_order_reader.
    METHODS validate IMPORTING order TYPE zsales_order
                     RAISING   zcx_invalid_order.
ENDCLASS.
```
**Avoid**:
```abap
CLASS zcl_sales_order_processor DEFINITION
  PUBLIC
  CREATE PUBLIC.

  PUBLIC SECTION.
    DATA reader TYPE REF TO zif_sales_order_reader.
    METHODS process.
    METHODS validate.
    METHODS reserve_stock.
ENDCLASS.
```
**ATC**: code pal for ABAP — *Final Class*

---

## RULE: prefer-new-over-create-object
**Why**: `NEW` is the modern constructor expression, composes inside functional calls, and matches every other modern ABAP construct. `CREATE OBJECT` is a relic that requires a separate variable declaration and reads as if it were a 2010 program.
**Do**:
```abap
DATA(processor) = NEW zcl_sales_order_processor( reader = sales_order_reader
                                                 log    = application_log ).

response = NEW zcl_response( status = 200 body = payload ).
```
**Avoid**:
```abap
DATA processor TYPE REF TO zcl_sales_order_processor.
CREATE OBJECT processor
  EXPORTING reader = sales_order_reader
            log    = application_log.
```
**ATC**: code pal for ABAP — *CREATE OBJECT Statement Usage*
