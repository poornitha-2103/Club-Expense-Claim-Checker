# Club Expense Claim Checker

A multi-page desktop Python/Tkinter dashboard for extracting, checking and classifying club expense claims. It **does not make payments**. The faculty treasurer signs every payment after seeing the bill.

## Requirements

- Python 3.10 or newer
- Tkinter (included with most standard Python installations; on some Linux distributions install the OS package `python3-tk`)
- Optional: OpenAI API key for live LLM extraction

## Run in VS Code

1. Download and extract this project ZIP.
2. Open the extracted `Club_Expense_Claim_Checker` folder in VS Code.
3. Open **Terminal → New Terminal**.
4. (Recommended) Create and activate a virtual environment:
   - Windows: `python -m venv .venv` then `.venv\Scripts\activate`
   - macOS/Linux: `python3 -m venv .venv` then `source .venv/bin/activate`
5. Install the optional LLM dependency: `python -m pip install -r requirements.txt`
6. For live LLM extraction, set `OPENAI_API_KEY` in the terminal environment before launching. Copying `.env.example` to `.env` is a reminder/template only; this version reads environment variables and does not automatically load `.env`. **Never commit your `.env` file.**
7. Run: `python app.py`

The multi-page GUI opens with Overview, Claims, Budgets, Reports and Guide pages. No input CSV is loaded automatically.

### Loading your own files

- Click **Choose claims CSV** beside **Input Claims CSV** and select your claims file.
- Click **Choose budget CSV** beside **Input Budget CSV** and select your budget file.
- Click **Check Claims** to process the selected files.
- Use the search box and status filter to find claims. Double-click a row to open its detail popup.
- Click **Export Outputs** and choose a destination folder.

## Input CSV formats

Claims CSV must have these exact column names:

```csv
id,club,date,note
C01,Coding Club,2026-10-01,"Paid Rs. 1,250 for banner printing at Copy Point. Bill attached"
C03,Dance Club,2026-10-02,Auto fare to inter-college fest 350. No bill from the auto driver
```

Budget CSV must have:

```csv
category,limit_per_club
printing,2000
food,4000
equipment,10000
```

The budget file provides a limit **per club for each category**. The accepted categories are `printing`, `food`, `travel`, `prizes`, `equipment`, `costumes`, and `other`. If a category is omitted from the budget file, its limit is treated as zero. Notes containing commas must be quoted according to normal CSV rules.

## Decision rules (specification order)

1. Check facts: the extracted amount must be one of the numbers written in the note (for example, `1,250` is 1250); it is never calculated. A bill is true only when the note mentions a bill/receipt/invoice/ticket and does not say it is missing or coming later.
2. `needs_more_info`: no amount.
3. `needs_bill`: no bill.
4. `needs_approval`: amount above Rs. 5,000, or same club + category + date claims add up to more than Rs. 5,000 (possible split claim).
5. `over_budget`: accepting the claim would take that club's spending in that category over its limit.
6. `ok_to_pay`: none of the above. Only these claims increase the running budget total.

The order is significant. For split claims above Rs. 5,000, each claim in the matching club/category/date group is flagged for approval. Claims are processed in the input CSV order, which should be date order.

## Output files

Export creates all four required reports and a `claim_json/` directory:

- `payable.csv` — only `ok_to_pay` claims, for the treasurer's review/signature.
- `review.csv` — every claim not `ok_to_pay`, including claim ID, club, date, status, reason and original note.
- `results.csv` — every claim and its final status, decision reason and extracted fields.
- `summary.txt` — counts for all five statuses and every club's accepted spending against every category budget limit.
- `claim_json/C01.json`, etc. — one JSON file per claim, containing exactly `amount`, `category`, `bill`, and `summary` as required by the specification.

## LLM extraction and fallback

With `OPENAI_API_KEY` set, the app calls the OpenAI chat completions API once per claim using the prompt in `system_prompt.txt` (same five-role structure: Authority, Exemplar, Constraint, Rubric, Metadata). The default model is `gpt-4o-mini`; override it with `OPENAI_MODEL`.

If no key is configured, the API errors, or an invalid response is returned, the app uses a conservative local extraction fallback so the workflow can still be demonstrated. The GUI detail popup and exported JSON show the final fact-checked extraction. For assessed submission, configure and test your own API key and review the generated outputs.

## Testing

Run the evaluation harness:

```bash
python test_app.py
```

It checks one example for each of the five statuses and three red-team cases: an instruction to pay immediately with no bill, a requested calculated amount, and two same-day split claims. It uses a deterministic fake extractor and does not require an API key.

## Project components included

- `app.py` — multi-page dashboard, CSV validation, extraction, fact checking, decision logic and report generation.
- `system_prompt.txt` — standalone five-role LLM prompt.
- `test_app.py` — five-status evaluation harness and three red-team tests.
- `project_specification.md` — concise implementation specification.
- `judgment_framework.md` — judgment framework and human checkpoint.
- `cost_latency_note.md` — API-call, latency and cost estimate template.
- `ai_decision_journal.md` and `reflection.md` — editable assessment templates.

## Important

This is a claim-checking aid, not an accounting authority. Review all classifications and source bills. The faculty treasurer remains the human checkpoint and signs every payment after seeing the bill. No payment is initiated by this software.