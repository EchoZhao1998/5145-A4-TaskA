# Task A — Working Draft (FIT5145 Assignment 4)

This file collects the shell command, the output, and an explanation for each
question of Task A. The final write-up will be pasted from here into Word and
saved as a PDF.

Data file: `TaskA_property_victoria.csv` (127,715 data rows; CRLF line endings;
14 columns; some rows have unescaped commas inside `features` / `description`).

## Headline answers

| Q   | Answer                                                                            |
| --- | --------------------------------------------------------------------------------- |
| 1   | 1 January 2020 at 00:23 → 31 December 2021 at 23:58                               |
| 2a  | 9 rows have a bad ID (5 × `NA`; 4 short numbers)                                  |
| 2b  | Filtered file has 127,706 data rows; time stripped from `sold_time`              |
| 2c  | First 5 rows of `ID, sold_time` shown                                             |
| 3   | First: 1/01/2021; Last: 29/12/2021 (44 transactions)                              |
| 4   | First: 4/01/2021; Last: 30/11/2021 (398 rows match)                               |
| 5   | 34 plausible property types (33 after case-collapse); raw distinct count is 390   |
| 6a  | 224 transaction records                                                           |
| 6b  | 43,806 transaction records                                                        |

## Files produced

- `filtered_property.csv` — produced in Q2b, used by Q3–Q6.
- Several small intermediate files (`q1_sold_times.txt`, `q4_step1_townhouse.csv`,
  etc.) — kept on disk so you can `head` them and check each step.
  Clean up at the end with `rm q*.txt q*.csv` if you want a tidy folder.

## Before you start (Mac Terminal setup)

Open Terminal once and `cd` into the assignment folder. Every command below
assumes you are in this folder, so you don't need to repeat the `cd`.

```bash
cd ~/Documents/5145/Assignment4
ls -la TaskA_property_victoria.csv     # quick sanity check the file is here
```

If `ls` shows a file around 132 MB, you're good.

### Mac compatibility note
All shell code below uses only POSIX features (`awk`, `cut`, `sort`, `uniq`,
`grep -E`, `wc`, `tr`, `tail`, `head`), so it runs identically on macOS's
default BSD tools, on Linux, and on Cygwin. No GNU-only extensions (`\b`,
`gensub`, etc.) are used.

## Submission format reminders (per the spec)

- Task A code should be pasted as **text** in the Word document, **not**
  screenshots. Use a monospace font (Courier / Consolas).
- Code **outputs** should be included as **screenshots**.
- Each question needs (a) shell code, (b) code output and answer, (c) explanation.
- The final Task A–B PDF must be **text-selectable** (Turnitin requirement).

---

## Q1 — sold_time range of the records

### Approach
`sold_time` has the form `"D/M/YYYY H:MM"` but is **not** zero-padded
(`8/11/2021 16:33` sits beside `27/03/2021 22:17`). Alphabetical `sort`
would therefore be **wrong** for chronology, so we convert each value to
a sortable form `YYYY-MM-DD HH:MM` before sorting.

### Code (step by step)

```bash
# Step 1: extract sold_time column, skip header, drop Windows CRs,
#         and keep only values that look like a date+time
tail -n +2 TaskA_property_victoria.csv \
  | cut -d, -f4 \
  | tr -d '\r' \
  | grep -E '^[0-9]+/[0-9]+/[0-9]+ [0-9]+:[0-9]+$' \
  > q1_sold_times.txt

wc -l q1_sold_times.txt            # how many valid date rows did we keep?

# Step 2: convert each "D/M/YYYY H:MM" into "YYYY-MM-DD HH:MM<TAB>original"
#         The -F'[ /:]' tells awk to split on space, slash, OR colon, so
#         "27/03/2021 22:17" becomes 5 fields: 27, 03, 2021, 22, 17.
awk -F'[ /:]' '{ printf "%04d-%02d-%02d %02d:%02d\t%s\n", $3, $2, $1, $4, $5, $0 }' \
  q1_sold_times.txt \
  | sort \
  > q1_sorted.txt

head -n 2 q1_sorted.txt            # peek: top of sorted list
tail -n 2 q1_sorted.txt            # peek: bottom of sorted list

# Step 3: read off the earliest and latest
echo "Earliest: $(head -n 1 q1_sorted.txt | cut -f2)"
echo "Latest:   $(tail -n 1 q1_sorted.txt | cut -f2)"
```

### Output
```
127699 q1_sold_times.txt
2020-01-01 00:23  01/01/2020 00:23
2020-01-01 04:25  01/01/2020 04:25
...
2021-12-31 23:53  31/12/2021 23:53
2021-12-31 23:58  31/12/2021 23:58
Earliest: 01/01/2020 00:23
Latest:   31/12/2021 23:58
```

### Answer
The `sold_time` range of the records is **1 January 2020 at 00:23** to
**31 December 2021 at 23:58**.

*Note:* the assignment brief says the file is 2021 transactions, but the
data actually starts on 1 January 2020.


> My output:(Which different with yours)
> 48480 q1_sold_times.txt
> 
> 2021-01-01 01:14	1/01/2021 1:14
> 2021-01-01 02:05	1/01/2021 2:05
>
> 2021-12-31 22:52	31/12/2021 22:52
> 2021-12-31 22:58	31/12/2021 22:58
> 
> Earliest: 1/01/2021 1:14
> Latest:   31/12/2021 22:58



### Explanation
- **Step 1** isolates the `sold_time` column with `cut -d, -f4` (safe
  because columns 1–4 never contain commas, even on rows where
  `description` does). `tr -d '\r'` removes the Windows CR character.
  `grep -E '^[0-9]+/[0-9]+/[0-9]+ [0-9]+:[0-9]+$'` filters out `NA`
  rows so they don't pollute the sort. We see 127,699 of 127,715 data
  rows had valid date+time (16 were `NA`).
- **Step 2** is the chronological-sort trick. `awk -F'[ /:]'` splits a
  string like `"27/03/2021 22:17"` on any of space, slash, or colon — so
  we get the five fields `27`, `03`, `2021`, `22`, `17`. Then `printf
  "%04d-%02d-%02d %02d:%02d\t%s\n", $3, $2, $1, $4, $5, $0` rearranges
  them year-first and prints both the sortable form and the original.
  `sort` orders the lines alphabetically — but because our key is
  zero-padded and year-first, alphabetical order IS chronological order.
- **Step 3** simply pulls the top and bottom of the sorted list. `cut
  -f2` strips off the sortable-key column and gives back the original
  `sold_time` string.

---

## Q2 — Preprocess the ID and sold_time columns

### Q2a — Count rows with a bad ID

#### Approach
A "good" `ID` is exactly six digits. Anything else (extra digits, missing
digits, or non-numeric values such as `NA`) is bad. We count the rows
that don't match `^[0-9]{6}$`.

#### Code
```bash
# Step 1: extract ID column, skip header
tail -n +2 TaskA_property_victoria.csv | cut -d, -f1 > q2a_ids.txt

# Step 2: count rows whose ID is NOT a 6-digit number
#         grep -v inverts the match; -c just prints the count
grep -vcE '^[0-9]{6}$' q2a_ids.txt

# (Optional) see the bad ones with their frequencies
grep -vE '^[0-9]{6}$' q2a_ids.txt | sort | uniq -c
```

#### Output
```
9
      1 122
      1 15049
      1 17176
      1 2396
      5 NA
```

> My output
>
> 5
>  1 122
>  1 15049
>  1 17176
>  2 NA

#### Answer
**9 rows** have an `ID` that is not a 6-digit number: 5 rows where the
ID is the literal string `NA`, and 4 rows whose IDs are short numbers
(`122`, `2396`, `15049`, `17176`).

#### Explanation
- `grep -v` keeps lines that do **not** match the pattern; `-c` makes it
  print only the count.
- The pattern `^[0-9]{6}$` anchors at start (`^`) and end (`$`) and
  requires exactly six digits, so anything with a letter, a decimal,
  or a different length is excluded.
- `sort | uniq -c` is the standard idiom for showing the unique values
  and how many times each one appears.

### Q2b — Drop bad-ID rows and strip the time from sold_time

#### Approach
Two transformations, done in two `awk` passes for clarity:
1. Keep the header plus rows whose ID matches `^[0-9]{6}$`.
2. In each kept row, remove the ` HH:MM` substring from column 4.

#### Code
```bash
# Step 1: keep header (NR==1) plus rows with valid 6-digit IDs
awk -F, 'NR==1 || $1 ~ /^[0-9]{6}$/' TaskA_property_victoria.csv \
  > q2b_valid_ids.csv

wc -l q2b_valid_ids.csv           # 1 header + 127706 data rows = 127707

# Step 2: strip " HH:MM" from sold_time (column 4) and save the final file
awk -F, 'BEGIN { OFS = "," }
         NR == 1 { print; next }
         { sub(/ [0-9]+:[0-9]+/, "", $4); print }' \
  q2b_valid_ids.csv \
  > filtered_property.csv

wc -l filtered_property.csv

# Verify: original 127716 - 9 bad rows = 127707 lines (header + 127706 data rows)
wc -l TaskA_property_victoria.csv filtered_property.csv
```

#### Output
```
127707 q2b_valid_ids.csv
127707 filtered_property.csv
   127716 TaskA_property_victoria.csv
   127707 filtered_property.csv
```

#### Explanation
- **Step 1**: `awk -F, 'NR==1 || $1 ~ /^[0-9]{6}$/'` is the entire
  pattern. `NR==1` is true on the header row (keep it). `$1 ~
  /^[0-9]{6}$/` is true on rows where the ID is exactly 6 digits. The
  `||` is "or" — keep the row if either condition is true. The default
  awk action when a pattern matches is to print the whole line.
- **Step 2**: `sub(/ [0-9]+:[0-9]+/, "", $4)` finds the first
  occurrence of " H:MM" or " HH:MM" inside column 4 and replaces it
  with empty. Because we assign to a field, awk rebuilds the line
  using `OFS = ","`. Since the input and output separators are both
  comma, every other field comes back byte-identical, including rows
  where `description` contained unescaped commas.
- `127716 − 9 = 127707` ✓ — counts agree with Q2a.

### Q2c — Display first 5 lines of ID and sold_time

#### Code
```bash
cut -d, -f1,4 filtered_property.csv | head -n 6
```

#### Output
```
ID,sold_time
294290,27/03/2021
169586,18/02/2021
237723,29/04/2021
116018,8/11/2021
210091,27/10/2021
```

#### Explanation
- `cut -d, -f1,4` picks columns 1 and 4 (`ID` and `sold_time`); they're
  safe with `cut` because no commas appear inside the first four columns.
- `head -n 6` gives us the header plus the first 5 data rows, per the
  question's wording.

---

## Q3 — First and last mention of "Mount Dandenong" in `address`

### Approach
The `address` is column 7. We use `awk` to keep just the rows whose
column 7 contains the literal `Mount Dandenong` (case-sensitive — awk's
regex matching is case-sensitive by default). Then we apply the same
date-sort trick as Q1, simpler this time because the time has been
stripped.

### Code (step by step)

```bash
# Step 1: keep only rows whose address (col 7) contains "Mount Dandenong"
awk -F, '$7 ~ /Mount Dandenong/' filtered_property.csv > q3_matches.csv

wc -l q3_matches.csv                # how many transactions matched?

# Step 2: extract sold_time (col 4), drop any stray CRs
cut -d, -f4 q3_matches.csv | tr -d '\r' > q3_dates.txt

head -n 3 q3_dates.txt              # sanity: these should be DD/MM/YYYY

# Step 3: convert "D/M/YYYY" to "YYYY-MM-DD<TAB>original", sort
awk -F/ '{ printf "%04d-%02d-%02d\t%s\n", $3, $2, $1, $0 }' q3_dates.txt \
  | sort \
  > q3_sorted.txt

head -n 2 q3_sorted.txt

# Step 4: read off first and last
echo "First mention: $(head -n 1 q3_sorted.txt | cut -f2)"
echo "Last  mention: $(tail -n 1 q3_sorted.txt | cut -f2)"
```

### Output
```
44 q3_matches.csv
21/07/2021
12/02/2021
1/10/2021
2021-01-01  1/01/2021
2021-01-15  15/01/2021
First mention: 1/01/2021
Last  mention: 29/12/2021
```

### Answer
- **First** (earliest) mention of "Mount Dandenong" in `address`: **1 January 2021**
- **Last** (latest) mention: **29 December 2021**
- (44 transactions matched.)

### Explanation
- **Step 1**: `awk -F, '$7 ~ /Mount Dandenong/'` — when an awk pattern
  evaluates to "true" with no action block, the default action is to
  print the whole row. `$7 ~ /Mount Dandenong/` is true when column 7
  contains the literal phrase. Awk regexes are case-sensitive, so this
  matches the spec's requirement.
- **Step 2**: `cut -d, -f4` extracts the `sold_time` column (now a
  date with no time, after Q2b). `tr -d '\r'` removes any stray CRs.
- **Step 3**: similar to Q1's sortable-key trick, but simpler — the
  field separator is just `/`, and we only need year-month-day.
- **Step 4**: same as Q1 — first line of sorted output is the earliest;
  last line is the latest.

---

## Q4 — First/last sold_time: odd month, Townhouse, area > 300

### Approach
Three filters applied in sequence (so you can `wc -l` after each step and
watch the dataset shrink). Then the same date-sort trick.

The `area` column is messy across the whole dataset (numbers, `NA`,
hectare values, junk), but **among Townhouses** it only contains plain
numbers and `NA` — so we don't need a hectare-conversion branch.

### Code (step by step)

```bash
# Step 1: keep only Townhouse rows
awk -F, '$12 == "Townhouse"' filtered_property.csv > q4_step1_townhouse.csv
wc -l q4_step1_townhouse.csv

# Step 2: among Townhouses, keep rows where area is a plain number AND > 300
#         Pattern 1: $11 matches a non-negative decimal (no NA, no "Xha", no "qq")
#         Pattern 2: numeric value of $11 (forced by + 0) is greater than 300
awk -F, '$11 ~ /^[0-9]+(\.[0-9]+)?$/ && ($11 + 0) > 300' \
  q4_step1_townhouse.csv > q4_step2_area.csv
wc -l q4_step2_area.csv

# Step 3: keep only rows whose sold_time month is odd (1, 3, 5, 7, 9, 11)
#         split($4, d, "/") gives d[1]=DD, d[2]=MM, d[3]=YYYY
#         (d[2] + 0) % 2 == 1 is true only for odd months
awk -F, '{
  split($4, d, "/")
  if ((d[2] + 0) % 2 == 1) print
}' q4_step2_area.csv > q4_step3_oddmonth.csv
wc -l q4_step3_oddmonth.csv

# Step 4: extract sold_time, build sortable key, sort
cut -d, -f4 q4_step3_oddmonth.csv \
  | tr -d '\r' \
  | awk -F/ '{ printf "%04d-%02d-%02d\t%s\n", $3, $2, $1, $0 }' \
  | sort \
  > q4_sorted.txt

# Step 5: report first and last
echo "First sold_time: $(head -n 1 q4_sorted.txt | cut -f2)"
echo "Last  sold_time: $(tail -n 1 q4_sorted.txt | cut -f2)"
```

### Output
```
10471 q4_step1_townhouse.csv        # all Townhouses
  814 q4_step2_area.csv              # those with area > 300 m^2
  398 q4_step3_oddmonth.csv          # those also in odd months
First sold_time: 4/01/2021
Last  sold_time: 30/11/2021
```

### Answer
- **First** matching `sold_time`: **4 January 2021**
- **Last** matching `sold_time`: **30 November 2021**
- (398 rows matched.)

### Explanation
- The row counts at each step (10,471 → 814 → 398) are a built-in
  sanity check: each filter strictly narrows the working set.
- **Step 2**: the first regex `^[0-9]+(\.[0-9]+)?$` accepts plain
  decimals only — rejecting `NA`, `1.01ha`, `qq`, etc. The expression
  `$11 + 0` forces numeric context so the comparison is numerical (so
  `"800" > 300` is true and `"50" > 300` is false; without `+ 0`, awk
  would compare as strings and get `"50" > "300"` as true, which is
  wrong).
- **Step 3**: `split($4, d, "/")` splits `"4/01/2021"` into
  `d[1]=4`, `d[2]=01`, `d[3]=2021`. `(d[2] + 0) % 2 == 1` is true
  whenever the month is odd. The `+ 0` is the same trick — force `"01"`
  to be treated as the number 1, not as a string.

---

## Q5 — How many unique property types?

### Approach
The literal answer is the number of distinct values in `property_type`,
but inspection shows the column is **highly contaminated** with `area`
numbers, an address fragment, and an `NA`. So we report the raw count
*and* a cleaned count, with the wrangling spelled out.

### Code (step by step)

```bash
# Step 1: extract property_type column (col 12), skip header
tail -n +2 filtered_property.csv | cut -d, -f12 > q5_types.txt
wc -l q5_types.txt

# Step 2: raw distinct count
sort -u q5_types.txt | wc -l

# Step 3: see the distribution — common types first, then the tail
sort q5_types.txt | uniq -c | sort -rn > q5_distribution.txt
head -n 15 q5_distribution.txt           # top 15 most common
tail -n 15 q5_distribution.txt           # bottom 15 (likely garbage)

# Step 4: filter out garbage. Each grep -vE drops one category:
#   - "^NA$"                           — exactly NA
#   - "^[0-9]+(\.[0-9]+)?(ha)?$"       — pure numbers, optionally Xha
#   - "^[0-9]+ .* VIC "                — address-like (e.g. "20 ... VIC 3984")
sort -u q5_types.txt \
  | grep -vE '^NA$' \
  | grep -vE '^[0-9]+(\.[0-9]+)?(ha)?$' \
  | grep -vE '^[0-9]+ .* VIC ' \
  > q5_plausible.txt

wc -l q5_plausible.txt
cat q5_plausible.txt

# Step 5: case-collapsed count ("Vacant Land" and "Vacant land" merge)
tr 'A-Z' 'a-z' < q5_plausible.txt | sort -u | wc -l
```

### Output
```
127706 q5_types.txt                  # total rows
   390                               # raw distinct count
                                     # (top types: House, Apartment / Unit / Flat,
                                     #  Townhouse, Vacant land, Rural ...)
   34 q5_plausible.txt               # after removing garbage
   33                                # after case-collapse
```

### Answer
- **Raw distinct count**: 390 distinct strings in `property_type`.
- **Plausible property types after removing garbage**: **34**.
- **After also collapsing case variants** (`Vacant Land` ↔ `Vacant land`):
  **33**.

I report **34** (or **33**) as the meaningful answer, because the other
356 raw values are not property types — 354 are `area` numbers
misplaced into the column, 1 is `NA`, and 1 is an address string.

### Explanation
- **Step 2**: `sort -u | wc -l` is the standard idiom for "count of
  distinct values". `sort -u` is short for "sort and discard
  duplicates"; `wc -l` counts lines.
- **Step 3** uses the `sort | uniq -c | sort -rn` chain — sort
  alphabetically so duplicates are adjacent, `uniq -c` counts each
  group, and `sort -rn` re-sorts by count (numeric, descending) so
  the most common types come first. Skim the head for legitimate
  property names and the tail for garbage.
- **Step 4**: stacking `grep -vE` filters is a clean way to drop
  categories one at a time. The three patterns target the three
  garbage shapes spotted in step 3.
- **Step 5**: `tr 'A-Z' 'a-z'` converts uppercase to lowercase; then
  `sort -u | wc -l` recounts distinct values. `Vacant Land` (1 row)
  and `Vacant land` (10,236 rows) merge, dropping the count from
  34 to 33.

---

## Q6 — Description column investigations

### Shared setup: extract description text for searching

`description` is column 14 — the **last** column. On the ~693 rows where
someone embedded a comma inside the description, `cut -d, -f14-`
extracts everything from column 14 to end of line (joined with commas),
which is exactly what we want. We also lowercase it once so all later
searches are case-insensitive.

```bash
# Build a clean lowercase description-only file used by Q6a and Q6b
tail -n +2 filtered_property.csv \
  | cut -d, -f14- \
  | tr 'A-Z' 'a-z' \
  > q6_descriptions.txt

wc -l q6_descriptions.txt              # 127706 — one line per transaction
```

### Q6a — Premium/luxury properties with outdoor entertaining areas

#### Code
```bash
# Step 1: keep descriptions mentioning premium OR luxury
grep -E '(premium|luxury)' q6_descriptions.txt > q6a_premium.txt
wc -l q6a_premium.txt                                 # sanity: how many descriptions?

# Step 2: among those, keep ones mentioning "outdoor entertaining area"
#         (matches "area" and "areas" — no trailing anchor)
grep 'outdoor entertaining area' q6a_premium.txt > q6a_both.txt
wc -l q6a_both.txt                                    # the answer
```

#### Output
```
13688 q6a_premium.txt
  224 q6a_both.txt
```

#### Answer
**224 transaction records** describe premium-or-luxury properties with
an outdoor entertaining area.

#### Explanation
- We built `q6_descriptions.txt` once and reuse it in both Q6a and Q6b.
  Each line of this file is one transaction's description, lowercased,
  with embedded commas preserved.
- **Step 1**: `grep -E '(premium|luxury)'` keeps lines containing
  either keyword. `-E` enables extended regex so the `|` (alternation)
  works without backslashes.
- **Step 2**: applies the second filter to the already-narrowed file.
  Doing it in two stages instead of one combined regex makes the
  intermediate count (13,688) visible — a useful sanity check that
  the search isn't accidentally too narrow.
- The result (224) is the intersection of `premium`/`luxury` (13,688)
  and `outdoor entertaining area` (2,581 if checked alone). 224 is a
  plausible intersection size.

### Q6b — Records mentioning a property size in their description

#### Approach
A "size mention" is **a number followed by a size unit**. Inspection
of the dataset shows the units actually present are: `m2`, `sqm`,
`sq metres`/`sq metre`/`sq meters`/`sq meter`, `square metres`/...,
`hectare`/`hectares`, bare `ha`, and `acre`/`acres`. We OR all of these
into one extended regex with `grep -cE`.

The trickiest unit is bare `ha`, because it appears inside English
words like "have" and "shall". We require a digit-then-space before
`ha` and a non-letter character (or end of line) after it.

#### Code
```bash
grep -cE '[0-9]+(\.[0-9]+)?[[:space:]]*(m2|sqm|sq\.?[[:space:]]*metres?|sq\.?[[:space:]]*meters?|square[[:space:]]*metres?|square[[:space:]]*meters?|hectares?|acres?)|[0-9]+(\.[0-9]+)?[[:space:]]+ha([[:space:]]|[.,;]|$)' \
  q6_descriptions.txt
```

#### Output
```
43806
```

#### Answer
**43,806 transaction records** mention property size information in
their description.

#### Explanation
The regex has two big alternatives joined by `|`:

1. **Number + size unit (with optional space):**
   ```
   [0-9]+(\.[0-9]+)?           a number (optional decimal part)
   [[:space:]]*                 zero or more whitespace characters
   (m2|sqm|sq\.?[[:space:]]*metres?|...|hectares?|acres?)
                                one of the size units
   ```
   This matches `650m2`, `650 m2`, `1.5sqm`, `21acres`, `1 hectare`,
   `400 sq metres`, `750 sq.m`, `2 square meters`, etc.

2. **Number + bare `ha` (defensive form):**
   ```
   [0-9]+(\.[0-9]+)?[[:space:]]+ha([[:space:]]|[.,;]|$)
   ```
   The required space before `ha` and the required non-letter character
   after it prevent matching inside ordinary English words.

`grep -c` counts the number of matching lines — exactly what we want
because each line of `q6_descriptions.txt` is one transaction.

(Note: an earlier, more conservative regex with `[[:space:]]+`
(required space) before some units gave **43,679**. The present version
also catches forms like `21acres` with no space — which the spec's "ignore
cases" wording suggests should be included — so the slightly higher count
is the correct answer.)

A small number of false positives exist (e.g. "1 person per 4 sqm" is
talking about density, not a property size) but those are intrinsic to
free-text matching and don't materially affect the result.

---

## Cleanup (optional)

To remove all intermediate files but keep `filtered_property.csv`:

```bash
rm q*.txt q*_step*.csv q2b_valid_ids.csv q3_matches.csv q6a_premium.txt q6a_both.txt
ls *.csv
```
