# Cost and Latency Note

The live LLM mode makes **one API call per claim**. Therefore one day containing N claims requires approximately N extraction calls (plus no additional model calls for Python validation, budget checks or report writing).

Latency depends on network conditions, provider load, model and note length. For planning, allow roughly 1–5 seconds per claim call; a batch of 20 sequential claims may therefore take around 20–100 seconds, excluding occasional retries. This is an estimate, not a service guarantee.

Cost estimate:
- Formula: `number_of_claims × (input_tokens × input_price + output_tokens × output_price)`.
- Actual pricing changes by model/provider and should be checked on the provider's current pricing page before submission.
- The app defaults to `gpt-4o-mini`; record the model, date, token usage and actual provider price used in your showcase report.
- Offline fallback mode makes zero API calls and has no model API cost.

The GUI performs processing on a background thread so the window remains responsive. The API key is read from the environment and is not stored in source code.