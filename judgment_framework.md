# Judgment Framework

| Component | What can go wrong? | What does the tool do? | What must a human judge? |
|---|---|---|---|
| LLM extraction | Amount/category/bill may be misread or prompt-injected | Constrained JSON extraction; claim note treated as untrusted evidence | Compare extracted facts with original note and source bill |
| Validation | Malformed JSON or unsupported amount | Exact fields/types/allowed categories checked; fallback used on invalid output | Investigate ambiguous or conflicting evidence |
| Fact checking | Model invents a total or claims a missing bill is attached | Amount must match a number in the note; bill flag checked against wording | Confirm actual bill/receipt/invoice/ticket exists |
| Classification | Wrong priority or threshold application | Applies the specification's rules in order | Review needs_approval, needs_more_info and unusual claims |
| Budget checking | Unaccepted claims incorrectly consume budget | Only ok_to_pay amounts update `(club, category)` totals | Confirm budget file is current and correctly approved |
| Reporting | Missing or mislabelled claims | Writes all claims to results; splits payable/review; writes per-claim JSON | Reconcile reports with original submissions |

## Error trade-off

A claim wrongly marked `ok_to_pay` is the more serious error: it could be passed forward without the required review and could expose the club to an unsupported payment. A claim wrongly sent for review creates delay and extra work, but a human can resolve it before payment. Therefore the tool should be conservative when evidence is uncertain.

## Human checkpoint

The faculty treasurer signs every payment after seeing the bill. This application classifies claims only and does not make payments.