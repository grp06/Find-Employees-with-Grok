# Find Employees with Grok

Agent Skill for using Grok inside Codex to find employees at supplied companies who match a supplied role or role family, then export direct X/Twitter handles to a CSV.

## Before You Start

Open the in-app browser inside Codex and log in to [grok.com](https://grok.com) before invoking the skill. The workflow depends on the authenticated Grok UI being available in the Codex in-app browser.

## Skill Location

The skill is stored at:

```text
.agents/skills/find-employees-with-grok/SKILL.md
```

It has also been installed locally at:

```text
/Users/georgepickett/.agents/skills/find-employees-with-grok/SKILL.md
```

## Example Prompt

```text
Use $find-employees-with-grok to find developer relations employees for:

Baseten
Databricks
Vercel

Write the output to devrel_x_handles.csv.
```

## Inputs

- `role_query`: the role or role family to search for, such as `developer relations`, `developer advocate`, `AI engineer`, or `founder`.
- `companies`: a newline- or comma-separated company list.
- `output_csv`: optional. Defaults to `grok_employee_x_handles.csv`.

## Outputs

- A CSV with header `company,x_handle`.
- A status ledger with one line per company, including row count and caveats.

The skill appends and validates after each company so partial progress survives browser or tool interruptions.
