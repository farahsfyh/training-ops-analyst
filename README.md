# Training Operations Analyst: Enterprise Agentic AI Capstone

A production-style, **multi-agent** assistant for AITraining2U training operations, built with Google ADK
(gemini-3.5-flash), MCP-style tool discovery and Streamlit.

**Live app:** <your-app>.streamlit.app

## Architecture summary
A `training_ops_coordinator` agent routes each question to one or both specialists exposed as `AgentTool`s:
`data_analyst_agent` (database) and `policy_agent` (policy markdown). Neither specialist has a hard-coded schema.
They use MCP discovery primitives (`mcp_list_tables`, `mcp_describe_table`, `mcp_read_query`, `mcp_list_policies`,
`mcp_read_policy`) at runtime, so a table added to the database is queryable immediately with zero code changes
(see the *Live DB evolution* demo). No mutation tool is ever registered (protocol-level least privilege).

**Two-tier guardrails.** *Input level* (`before_model_callback`, attached to every agent): PII (emails, Malaysian
mobile numbers, MyKad, cards) is masked in place before the LLM sees it, and prompt-injection / destructive intents
are rejected before any tokens are spent. *Data level*: `is_sql_mutation` parses every query with `sqlparse` and
allows exactly one SELECT (no DDL/DML/PRAGMA/stacked statements), and every connection is opened with
`file:training_ops.db?mode=ro`, so writes are physically impossible. Answers are PII-masked again on output.

**Observability and evaluation.** `after_model_callback` records tokens and estimated USD per agent. A golden dataset
of 6 cases (data, policy, mixed, safety) is graded by an LLM-as-a-Judge (1-5); see `evaluation_report.md`.

## Run locally
```
pip install -r requirements.txt
streamlit run app.py        # needs GOOGLE_API_KEY in env or .streamlit/secrets.toml
```
