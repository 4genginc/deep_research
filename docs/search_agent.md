# Understanding `search_agent.py`

This guide walks through how the search agent works. It explains the role of each constant and how the agent is configured so you can tweak it for your own research pipeline.

## Purpose

`search_agent.py` defines a single agent responsible for executing one web search and summarizing the results. The agent queries the web and returns a concise description of what it finds.

## Key Components

### 1. Imports
```python
from agents import Agent, WebSearchTool, ModelSettings
```
* **Agent** – core class used to wrap a language model with tools and instructions.
* **WebSearchTool** – a built in tool that performs an internet search.
* **ModelSettings** – allows additional options when calling the model.

### 2. Instructions
```python
INSTRUCTIONS = (
    "You are a research assistant. Given a search term, you search the web for that term and "
    "produce a concise summary of the results. The summary must 2-3 paragraphs and less than 300 "
    "words. Capture the main points. Write succintly, no need to have complete sentences or good "
    "grammar. This will be consumed by someone synthesizing a report, so its vital you capture the "
    "essence and ignore any fluff. Do not include any additional commentary other than the summary itself."
)
```
These instructions tell the LLM to search the web and summarize the findings in under 300 words. It emphasizes brevity and discourages any extra commentary.

### 3. Search Agent
```python
search_agent = Agent(
    name="Search agent",
    instructions=INSTRUCTIONS,
    tools=[WebSearchTool(search_context_size="low")],
    model="gpt-4o-mini",
    model_settings=ModelSettings(tool_choice="required"),
)
```
* **name** – Label used in traces and logs.
* **tools** – Uses a `WebSearchTool` with minimal context to keep searches fast.
* **model** – The GPT‑4o mini model powers the agent.
* **model_settings** – `tool_choice="required"` forces the LLM to call the search tool.

## Example Usage
The agent is used in `ResearchManager.search` for each planned search item:
```python
input = f"Search term: {item.query}\nReason for searching: {item.reason}"
result = await Runner.run(
    search_agent,
    input,
)
summary = str(result.final_output)
```
Given a search query and its reasoning, the agent returns the summary text.

## Extending or Modifying
* **Adjust summary length** – edit the `INSTRUCTIONS` string.
* **Change search depth** – modify `search_context_size`.
* **Swap models** – update the `model` parameter.
* **Allow optional tool use** – remove `tool_choice="required"` if desired.

## Summary
`search_agent.py` is small but critical. It encapsulates how the system performs individual web searches and condenses the results. Modify this file to tune search behavior or the summarization style.

