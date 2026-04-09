# RPT6000 – Year-To-Date Sales Report

**Course:** COBOL Programming – Chapter 6 Assignment
**Author:** [Gabe Dilley](https://github.com/gawdilley)
**Date:** April 2, 2026
**GitHub:** [COBOL-Chapter-6-Assignment](https://github.com/gawdilley/COBOL-Chapter-6-Assignment)

---

## Description

RPT6000 is a COBOL batch reporting program that reads a customer master file (`CUSTMAST`) alongside a sales rep reference file (`SALESREP`) and produces a formatted Year-To-Date (YTD) Sales Report (`RPT6001`). The key advancement in this version is the introduction of an in-memory lookup table — the sales rep file is loaded into a table at startup and searched during report generation to print each rep's name on the customer detail line, eliminating the need to match records sequentially.

---

## What the Program Does

### Input
The program reads two input files:

**Customer Master (`CUSTMAST`)** — one record per customer containing:
- Branch number
- Sales rep number
- Customer number
- Customer name
- Sales amount for the current year-to-date
- Sales amount for the prior year-to-date

**Sales Rep Reference (`SALESREP`)** — one record per sales rep containing:
- Sales rep number
- Sales rep name

### Processing
The program performs the following steps:

1. **Initializes** the sales rep table in working storage before opening any files.
2. **Opens** all three files — two input files and one output file.
3. **Loads the sales rep table** by reading the entire `SALESREP` file into a 100-element table in working storage, storing the rep number and name for later lookup.
4. **Reads** each customer record using a `WITH TEST AFTER` loop, ensuring at least one pass occurs before checking for end-of-file.
5. **Detects control breaks** at two levels using an `EVALUATE TRUE` structure:
   - **Sales rep break** — prints a sales rep total line and resets sales rep accumulators.
   - **Branch break** — prints both a sales rep total and a branch total line before continuing.
6. **Looks up the sales rep name** via a `SEARCH` statement against the in-memory table whenever a new rep appears on the report. If the rep number is not found, `UNKNOWN` is printed in place of the name.
7. **Suppresses repeated fields** — when the branch or sales rep hasn't changed, those columns are left blank on the detail line for readability.
8. **Calculates** for each customer, each sales rep, each branch, and the grand total:
   - **Change Amount** — the difference between this year's and last year's YTD sales.
   - **Change Percent** — displayed using a `+++9.9` sign-leading edit pattern. If last year's sales are zero, `N/A` is displayed; if the result overflows the field, `OVRFLW` is displayed.
9. **Accumulates** running totals at three levels:
   - **Sales rep totals** — reset via `INITIALIZE` after each sales rep total line is printed.
   - **Branch totals** — reset via `INITIALIZE` after each branch total line is printed.
   - **Grand totals** — accumulated from branch totals across the entire run.
10. **Manages pagination** — when the line count exceeds 55 lines, a new page is started with fresh heading lines.
11. **Uses `PACKED-DECIMAL`** storage for all numeric print and total fields to improve computational efficiency.

### Output
The printed report (`RPT6001`) includes:

- **Report headings** on each page with the current date, time, report title, report ID, and page number.
- **Column headers** for Branch #, Sales Rep #, Sales Rep Name, Customer #, Customer Name, Sales This YTD, Sales Last YTD, Change Amount, and Change Percent.
- **One detail line per customer**, with branch number, sales rep number, and sales rep name suppressed when unchanged from the previous line.
- **Sales rep total lines** (marked with `*`) after the last customer for each sales rep, with currency-sign (`$`) edited output.
- **Branch total lines** (marked with `**`) after the last sales rep in each branch, with currency-sign edited output.
- **A grand total line** (marked with `***`) at the end of the report.

---

## Example Output

```
DATE:  04/02/2026                    YEAR-TO-DATE SALES REPORT            PAGE:   1
TIME:  10:30                                                               RPT6001

                                                    SALES       SALES        CHANGE    CHANGE
BRANCH  SALESREP               CUSTOMER            THIS YTD    LAST YTD     AMOUNT    PERCENT
------  -------------  --------------------------  -----------  -----------  -----------  -------
 01      01 JOHNSON    00101 ACME CORPORATION        15,000.00-   12,500.00-   2,500.00-    +20.0
                       00102 SMITH HARDWARE           8,200.00-    9,100.00-     900.00      -9.9
                       00103 METRO SUPPLY CO          4,750.00-    3,200.00-   1,550.00-    +48.4
                              SALESREP TOTAL        $27,950.00-  $25,800.00-  $2,150.00-    +8.3 *

         02 MARTINEZ   00201 LAKE CITY TOOLS         22,000.00-   19,800.00-   2,200.00-    +11.1
                       00202 RIVERSIDE MFG            9,100.00-    9,100.00-        .00      +0.0
                              SALESREP TOTAL        $31,100.00-  $28,900.00-  $2,200.00-     +7.6 *

                             BRANCH TOTAL          $$$59,050.00-  $54,700.00-  $4,350.00-    +7.9 **

 02      01 JOHNSON    00301 JONES ELECTRIC          11,500.00-   11,500.00-        .00      +0.0
                       00302 WESTERN TOOLS            6,800.00-    5,400.00-   1,400.00-    +25.9
                              SALESREP TOTAL        $18,300.00-  $16,900.00-  $1,400.00-     +8.3 *

                             BRANCH TOTAL          $$$18,300.00-  $16,900.00-  $1,400.00-    +8.3 **

                          GRAND TOTAL              77,350.00    71,600.00     5,750.00       8.0 ***
```

---

## New Concepts Used

- **Table loading from a file** — reading an entire reference file (`SALESREP`) into a working-storage table at program startup using a `PERFORM VARYING` loop before the main processing begins
- **`OCCURS` clause** — defining a 100-element repeating group (`SALESREP-GROUP`) in working storage to hold the in-memory sales rep table
- **`INDEXED BY`** — declaring a table index (`SRT-INDEX`) alongside the `OCCURS` clause and using `SET` to initialize and control it during the table search
- **`SEARCH` statement** — performing a sequential table lookup to match the current customer's sales rep number against the loaded table, with an `AT END` fallback of `UNKNOWN` when no match is found
- **`COPY` statement** — referencing external copybook members (`CUSTMAST` and `SALESREP`) for the `FD` record layouts instead of defining them inline, promoting reusability
- **`INITIALIZE` statement** — resetting accumulator fields to zero after printing each total line, replacing the explicit `MOVE ZERO TO` pattern used in prior assignments
- **`PACKED-DECIMAL` storage** — declaring all `PRINT-FIELDS`, `TOTAL-FIELDS`, and `CALCULATED-FIELDS` groups with `PACKED-DECIMAL` usage to store numeric data in compressed BCD format for more efficient arithmetic
- **`+++9.9` sign-leading edit pattern** — using a floating plus/minus sign on the change percent field of the customer line, replacing the trailing `-` edit character used in earlier reports
- **`REDEFINES` for alternate field display** — using `CL-CHANGE-PERCENT-R REDEFINES CL-CHANGE-PERCENT` (and similar patterns on total lines) to allow the same output field to be written as either a formatted number or a text string like `N/A` or `OVRFLW`
- **`$$,$$$$9.99-` currency editing** — applying floating dollar-sign edit patterns on the sales rep and branch total lines to produce professional currency-formatted output
- **Two-file input** — opening and managing two separate input files simultaneously, with independent EOF switches and read routines for each

---

## Author

| Name | Profile |
|------|---------|
| Gabe Dilley | [GitHub](https://github.com/gawdilley) |
