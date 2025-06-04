# Understanding `writer_agent.py`

This guide explains how the writer agent synthesizes research findings into a full report. It details the core components so you can modify the behavior or integrate the agent into your own pipeline.

## Purpose

`writer_agent.py` defines an agent that takes the original query and a list of search summaries and produces a long‑form markdown report. It also returns a short summary and suggested follow‑up questions.

## Key Components

### 1. Imports
```python
from pydantic import BaseModel, Field
from agents import Agent
```
* **Pydantic** provides the `BaseModel` and `Field` classes used to define the output schema.
* **Agent** is the wrapper from the custom framework that runs an LLM with given instructions.

### 2. Instructions
```python
INSTRUCTIONS = (
    "You are a senior researcher tasked with writing a cohesive report for a research query. "
    "You will be provided with the original query, and some initial research done by a research assistant.\n"
    "You should first come up with an outline for the report that describes the structure and "
    "flow of the report. Then, generate the report and return that as your final output.\n"
    "The final output should be in markdown format, and it should be lengthy and detailed. Aim "
    "for 5-10 pages of content, at least 1000 words."
)
```
These instructions tell the LLM to plan an outline and then write a comprehensive markdown document of at least 1000 words.

### 3. Data Model
```python
class ReportData(BaseModel):
    short_summary: str
    markdown_report: str
    follow_up_questions: list[str]
```
* **short_summary** – a two to three sentence recap of the findings.
* **markdown_report** – the full report text in markdown format.
* **follow_up_questions** – additional topics that could be researched next.

### 4. The Writer Agent
```python
writer_agent = Agent(
    name="WriterAgent",
    instructions=INSTRUCTIONS,
    model="gpt-4o-mini",
    output_type=ReportData,
)
```
* **name** – label for logging and tracing.
* **instructions** – the prompt shown above.
* **model** – uses the GPT‑4o mini variant.
* **output_type** – enforces that the response conforms to `ReportData`.

Calling `writer_agent` through `Runner.run()` yields a validated `ReportData` instance.

## Example Usage
The writer agent is used in `ResearchManager.write_report`:
```python
input = f"Original query: {query}\nSummarized search results: {search_results}"
result = await Runner.run(
    writer_agent,
    input,
)
report = result.final_output_as(ReportData)
```
Given the original query and all search summaries, the agent returns the finished markdown report along with a short summary and follow‑up questions.

## Extending or Modifying
* **Change report length or style** – edit the `INSTRUCTIONS` string.
* **Switch to a different model** – modify the `model` parameter.
* **Capture more metadata** – extend `ReportData` with additional fields.

## Summary
`writer_agent.py` is responsible for transforming raw search summaries into a polished research report. Understanding this file lets you customize the final output of the Deep Research pipeline.

