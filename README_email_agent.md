# Understanding `email_agent.py`

This guide details how the email agent formats the final report and sends it via SendGrid. Read this if you want to customize how reports are delivered to users.

## Purpose

`email_agent.py` defines an agent that converts the markdown report into HTML and emails it. The agent exposes a single tool, `send_email`, which uses the SendGrid API.

## Key Components

### 1. Imports
```python
import os
from typing import Dict
import sendgrid
from sendgrid.helpers.mail import Email, Mail, Content, To
from agents import Agent, function_tool
```
* **os** – reads the `SENDGRID_API_KEY` from environment variables.
* **sendgrid** and **helpers.mail** – used to construct and send the email.
* **Agent** and **function_tool** – from the custom framework, enabling the LLM to call the tool.

### 2. `send_email` function
```python
@function_tool
def send_email(subject: str, html_body: str) -> Dict[str, str]:
    """Send an email with the given subject and HTML body"""
    sg = sendgrid.SendGridAPIClient(api_key=os.environ.get('SENDGRID_API_KEY'))
    from_email = Email("rj.geng@gmail.com")  # your verified sender
    to_email = To("aiai_yang@4g-eng.com")    # recipient address
    content = Content("text/html", html_body)
    mail = Mail(from_email, to_email, subject, content).get()
    response = sg.client.mail.send.post(request_body=mail)
    print("Email response", response.status_code)
    return {"status": "success"}
```
* Decorated with `@function_tool` so the agent can invoke it.
* Uses SendGrid to send the HTML content.

### 3. Instructions
```python
INSTRUCTIONS = """You are able to send a nicely formatted HTML email based on a detailed report.
You will be provided with a detailed report. You should use your tool to send one email, providing the
report converted into clean, well presented HTML with an appropriate subject line."""
```
This prompt tells the model to transform the report into polished HTML and call `send_email` exactly once.

### 4. The Email Agent
```python
email_agent = Agent(
    name="Email agent",
    instructions=INSTRUCTIONS,
    tools=[send_email],
    model="gpt-4o-mini",
)
```
* **name** – shown in traces and logs.
* **instructions** – the prompt described above.
* **tools** – exposes only the `send_email` function.
* **model** – uses the GPT‑4o mini variant.

Calling `Runner.run` with this agent will return the same report after the email is sent.

## Example Usage
The agent is used in `ResearchManager.send_email`:
```python
result = await Runner.run(
    email_agent,
    report.markdown_report,
)
```
Given the markdown report text, the agent formats it as HTML and sends it to the configured recipient.

## Extending or Modifying
* **Update addresses** – change `from_email` and `to_email` in `send_email`.
* **Customize formatting** – modify the LLM instructions or post-process the HTML.
* **Alternate providers** – replace the SendGrid logic with another email service.

## Summary
`email_agent.py` handles the final delivery step of the research pipeline. Understanding this file lets you adjust how reports are formatted and where they are sent.
