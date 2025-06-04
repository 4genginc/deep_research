# Understanding `planner_agent.py`

This guide walks through the structure and purpose of the `planner_agent.py` module. It is intended for developers who want to understand how search planning works in the Deep Research system.

## Purpose

`planner_agent.py` defines a single agent responsible for turning a high‑level research question into specific search terms. The agent is powered by an LLM through the `agents` framework and outputs a list of search instructions.

The rest of the system uses these search terms to perform web searches, summarize results and eventually generate a detailed research report.

## Key Components

### 1. Imports
```python
from pydantic import BaseModel, Field
from agents import Agent
```
* **Pydantic** is used to define data models (`WebSearchItem` and `WebSearchPlan`).
* **Agent** comes from the custom `agents` framework. It wraps an LLM model with instructions and output typing.

### 2. Constants
```python
HOW_MANY_SEARCHES = 5
```
`HOW_MANY_SEARCHES` controls how many search terms the planner should output. By default the agent returns five different queries. Adjust this number if you want more or fewer searches.

```python
INSTRUCTIONS = f"You are a helpful research assistant..."
```
The `INSTRUCTIONS` string tells the language model how to behave. It explains that the agent should come up with web searches that best answer the user’s question and return exactly `HOW_MANY_SEARCHES` of them.

### 3. Data Models
```python
class WebSearchItem(BaseModel):
    reason: str
    query: str
```
Each search suggestion has a **reason** (why this search is useful) and the actual **query** string to send to a search engine.

```python
class WebSearchPlan(BaseModel):
    searches: list[WebSearchItem]
```
`WebSearchPlan` simply groups a list of `WebSearchItem` objects. It is the structured output produced by the planner agent.

### 4. The Planner Agent
```python
planner_agent = Agent(
    name="PlannerAgent",
    instructions=INSTRUCTIONS,
    model="gpt-4o-mini",
    output_type=WebSearchPlan,
)
```
* **name** – Label used in logs and traces.
* **instructions** – Prompt sent to the language model.
* **model** – Specifies which LLM variant to use.
* **output_type** – Tells the framework to parse the model response into a `WebSearchPlan` object.

Calling `planner_agent` via `Runner.run()` (see `research_manager.py`) will return a validated `WebSearchPlan` instance.

## Example Usage
The planner is invoked in `ResearchManager.plan_searches`:
```python
result = await Runner.run(
    planner_agent,
    f"Query: {query}",
)
plan = result.final_output_as(WebSearchPlan)
```
Given a user query, the agent returns a plan containing five search items. Each item includes the reasoning text and the search term.

## Extending or Modifying
* **Change the number of searches** – modify `HOW_MANY_SEARCHES`.
* **Adjust instructions** – edit the `INSTRUCTIONS` string for different behavior.
* **Switch models** – change the `model` parameter of `planner_agent`.
* **Add fields** – extend `WebSearchItem` or `WebSearchPlan` for additional metadata (e.g., priority).

## Summary
`planner_agent.py` is concise but central to the research pipeline. It converts a user’s question into concrete search tasks that other agents perform. Understanding this file helps you customize how the system explores any research topic.
