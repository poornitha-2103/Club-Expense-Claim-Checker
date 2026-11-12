# Club Expense Claim Checker — Implementation Specification

## Inputs
The user selects two external CSV files in the GUI; the program never assumes local sample files exist.
- Claims: `id,club,date,note` (at least 20 claims for the assessment; date order recommended).
- Budgets: `category,limit_per_club`, where the limit is per club for each category.

## Extraction JSON
For every claim, return exactly:
`{"amount": 1250, "category": "printing", "bill": true, "summary": "Banner printing"}`
- amount: number or null; a total explicitly written in the note, never calculated or read from words.
- category: printing, food, travel, prizes, equipment, costumes, other.
- bill: true or false; true only when bill/receipt/invoice/ticket is attached.
- summary: no more than 10 words.

## Fact checks and decision priority
1. Validate the amount against numbers written in the note (`1,250` equals 1250). Bill is true only if the note mentions a bill and does not say missing/coming later.
2. `needs_more_info`: no amount.
3. `needs_bill`: no bill.
4. `needs_approval`: amount above Rs. 5,000, or same club/category/date claims total more than Rs. 5,000.
5. `over_budget`: club/category accepted spending would exceed the category limit.
6. `ok_to_pay`: none of the above; add amount to that club/category running total.

Only `ok_to_pay` claims count toward budgets. Running totals use `(club, category)` tuple keys.

## Outputs
- `payable.csv`: only `ok_to_pay`.
- `review.csv`: all other claims, with ID, club, date, status, reason and original note.
- `results.csv`: all claims and final status.
- `summary.txt`: counts for all five statuses and each club's spending against each category limit.
- `claim_json/<claim-id>.json`: one extraction JSON per claim.

## Human checkpoint
The tool pays nothing. The faculty treasurer signs every payment after seeing the bill.