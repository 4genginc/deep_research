# Understanding `deep_research.py`

This guide explains how the `deep_research.py` file launches the Gradio user interface and ties the research pipeline together.

## Purpose

`deep_research.py` provides a minimal web front end. It loads environment variables, streams progress from the `ResearchManager`, and displays the final markdown report.

## Key Components

### 1. Imports
```python
import gradio as gr
from dotenv import load_dotenv
from research_manager import ResearchManager
```
* **gradio** – used to build the web interface.
* **load_dotenv** – loads API keys and other configuration from `.env`.
* **ResearchManager** – orchestrates the planning, searching, writing, and emailing steps.

### 2. Environment Setup
```python
load_dotenv(override=True)
```
This call ensures environment variables from `.env` are available before any agents run.

### 3. Async `run` Function
```python
async def run(query: str):
    async for chunk in ResearchManager().run(query):
        yield chunk
```
* Creates a new `ResearchManager` for each query.
* Yields chunks of text as the manager progresses through the pipeline, allowing the UI to stream updates.

### 4. Gradio UI
```python
with gr.Blocks(theme=gr.themes.Default(primary_hue="sky")) as ui:
    gr.Markdown("# Deep Research")
    query_textbox = gr.Textbox(label="What topic would you like to research?")
    run_button = gr.Button("Run", variant="primary")
    report = gr.Markdown(label="Report")

    run_button.click(fn=run, inputs=query_textbox, outputs=report)
    query_textbox.submit(fn=run, inputs=query_textbox, outputs=report)

ui.launch(inbrowser=True)
```
* Builds a simple form with a textbox and a button.
* Clicking **Run** (or pressing Enter) triggers the `run` function.
* The markdown component streams each yielded chunk until the final report appears.
* `ui.launch(inbrowser=True)` opens the interface in your default browser.

## Example Usage
Run the script directly:
```bash
python deep_research.py
```
Enter a query like "Future of renewable energy" and watch the progress messages followed by the full report.

## Extending or Modifying
* **Customize the theme** – adjust the `gr.themes.Default` parameters.
* **Add more status info** – display intermediate chunks in separate components or log them elsewhere.
* **Alternative front ends** – call `ResearchManager.run` from your own web framework or CLI app.

## Summary
`deep_research.py` is the entry point for users. It loads configuration, spins up a Gradio interface, and streams output from `ResearchManager`. Understanding this file helps you integrate the research pipeline into different UIs or automate it further.
