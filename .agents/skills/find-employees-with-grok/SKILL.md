---
name: find-employees-with-grok
description: Use when the user wants Codex to use Grok in the in-app browser to find employees at a supplied list of companies who match a supplied role or role family, then export direct X/Twitter handles to a CSV. Trigger for requests like "find developer relations people at these companies", "use Grok to find AI engineers at company list", or "make a company,handle CSV from Grok results".
---

# Find Employees With Grok

Use Grok as the research surface and produce a durable CSV of company-to-X-handle matches for a user-supplied role name or role family.

## Required Inputs

Collect or infer:

- `role_query`: the role or role family to search for, such as `developer relations`, `developer advocate`, `AI engineer`, or `founder`.
- `companies`: a newline- or comma-separated company list.
- `output_csv`: default to `grok_employee_x_handles.csv` in the current workspace.

If either `role_query` or `companies` is missing, ask for only the missing input.

## Browser Requirement

Use the Codex in-app browser through the Browser skill/plugin. Do not use Chrome unless the user explicitly asks for Chrome.

Before starting, ensure the user has opened the Codex in-app browser and is logged in to `grok.com`. If Grok is not open, open `https://grok.com`. If login, CAPTCHA, paywall, age gate, or account permission blocks the workflow, stop and ask the user to handle it.

## Output Files

Create or update:

- CSV: `company,x_handle`
- Status ledger: same basename plus `_status.md`

Example:

```csv
company,x_handle
Baseten,@philipkiely
Baseten,@Madisonkanna
```

Status ledger line format:

```markdown
- 2026-05-14T19:38:18Z | Baseten | rows_added=2 | caveat if any
```

## Query Template

For each company, send Grok one query:

```text
Find me all of the employees at [COMPANY] (which is a tech company) who are [ROLE_QUERY] or similar roles. Reply only with direct links to their X accounts. Do not provide explanations. Only reply with links.
```

Adapt only `[COMPANY]` and `[ROLE_QUERY]`. Keep the "direct links only" instruction intact.

## Workflow

1. Initialize the CSV and status ledger if missing.
2. Read the status ledger and skip companies already recorded, unless the user asks to rerun them.
3. For each remaining company:
   - Submit the query to Grok in the in-app browser.
   - Wait for the answer to finish or become stable enough to parse.
   - Extract only direct `x.com/<handle>` or `twitter.com/<handle>` profile links from Grok's answer.
   - Normalize each profile to `@handle`.
   - Exclude company accounts, Grok/X platform accounts, explanations, names, LinkedIn URLs, source buttons, and speculative handles not present as direct X/Twitter links in the answer.
   - Append new `(company,x_handle)` pairs, avoiding exact duplicates.
   - Save the CSV immediately after that company, even if zero rows were found.
   - Append one status ledger line for that company.
   - Validate the CSV before moving on.
4. Continue until every supplied company has a status entry.

## Parsing Rules

Valid handles must match:

```text
^@[A-Za-z0-9_]{1,15}$
```

Normalize links like:

- `https://x.com/someone` -> `@someone`
- `https://twitter.com/someone?lang=en` -> `@someone`

Ignore:

- URLs that are not X/Twitter profile URLs.
- Search URLs, status URLs, list URLs, posts, communities, or source citations.
- Handles mentioned only in biographies or explanations unless Grok's answer presents them as direct profile links.
- Duplicate handles for the same company.

## Validation

After each company, verify:

- CSV header is exactly `company,x_handle`.
- Every data row has exactly two columns.
- Every nonempty handle matches the valid-handle regex.
- The status ledger contains exactly one completion entry for that company unless the user asked to rerun.

At the end, report:

- Output CSV path.
- Status ledger path.
- Companies completed.
- Total rows.
- Companies with zero handles.
- Any caveats.
- First and last few CSV rows.

## Gotchas

- Grok pages can be heavy. Prefer short polling plus salvage: if a tool call times out, reconnect to the same in-app browser tab, inspect the latest visible answer, and append whatever was actually returned before continuing.
- Do not assume "no handles" from a timeout alone. Inspect the visible answer or DOM snapshot first.
- If Grok returns explanatory text despite the prompt, parse only direct profile links and note the caveat in the status ledger.
- If a company name is ambiguous, keep the user-supplied name in the CSV and note ambiguity in the ledger rather than inventing a different company label.
