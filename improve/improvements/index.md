# Improvements — open, prioritized (P0–P3), cited

* [The TypeScript agent caps resource text in the system prompt; the Python agent — the one that actually runs — does not](python-resource-text-uncapped.md) - chat_node serialises every downloaded resource's full page text into the system prompt on every turn with no truncation and no aggregate budget, so one long article can exceed an entire per-minute token allowance in a single request and 429 unrecoverably; the TS port fixed exactly this with MAX_RESOURCE_CHARS/MAX_TOTAL_RESOURCE_CHARS, but npm run dev starts the Python backend
