# Understanding `research_manager.py`

This guide explains how the `ResearchManager` orchestrates the entire multi-agent pipeline. It is intended for developers who want to tweak or extend the workflow.

## Purpose

`research_manager.py` coordinates the planner, search, writer, and email agents. It yields progress updates for a UI and ultimately returns the finished report.

## Key Components

### 1. Imports
```python
from agents import Runner, trace, gen_trace_id
from search_agent import search_agent
from planner_agent import planner_agent, WebSearchItem, WebSearchPlan
from writer_agent import writer_agent, ReportData
from email_agent import email_agent
import asyncio
```
* **Runner** executes an agent and returns structured output.
* **trace** and **gen_trace_id** hook into OpenAI tracing for debugging.
* Other imports bring in the four agents that perform the work.
* **asyncio** enables concurrent searches.

### 2. `ResearchManager` class

The class exposes several async methods that run the pipeline step by step.

#### `run(query: str)`
Yields status strings as each phase completes and finally yields the markdown report text.
1. Generates a unique trace ID and prints a link for debugging.
2. Calls `plan_searches` to get search terms.
3. Calls `perform_searches` concurrently for each term.
4. Calls `write_report` to synthesize a long markdown document.
5. Sends the report via `send_email`.

#### `plan_searches(query)`
Runs `planner_agent` to produce a `WebSearchPlan` containing multiple `WebSearchItem` objects.

#### `perform_searches(search_plan)`
Creates asyncio tasks for `search` and waits for them with `asyncio.as_completed`. Progress is printed after each task.

#### `search(item)`
Invokes `search_agent` with the query and reason from a `WebSearchItem`. If any search fails, it returns `None` so the pipeline continues.

#### `write_report(query, search_results)`
Aggregates all search summaries and calls `writer_agent` to produce the final `ReportData` object.

#### `send_email(report)`
Uses `email_agent` to format the report as HTML and deliver it via SendGrid.

## Example Usage
`deep_research.py` creates a simple Gradio UI that streams the output from `ResearchManager.run`:
```python
async for chunk in ResearchManager().run(query):
    yield chunk
```
The last chunk contains the full markdown report.

## Extending or Modifying
* **Change concurrency** – adjust how tasks are created in `perform_searches`.
* **Add logging** – insert log statements or integrate a logging library.
* **Custom post-processing** – modify `write_report` or `send_email` to store reports or send them elsewhere.

## Summary
`research_manager.py` ties all agents together. Understanding this orchestrator lets you customize how queries are planned, searched, and turned into professional reports.
