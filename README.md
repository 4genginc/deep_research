# Deep Research System 🔍

A sophisticated multi-agent research pipeline that automatically conducts comprehensive investigations on any topic using AI agents, web search, and intelligent synthesis.

## 🎯 What This System Does

The Deep Research System transforms a simple query into a comprehensive research report through an automated pipeline of specialized AI agents. Instead of manually searching, reading, and synthesizing information from multiple sources, this system:

1. **Plans** strategic searches based on your query
2. **Executes** multiple web searches concurrently  
3. **Synthesizes** findings into a professional report
4. **Delivers** the report via email in HTML format

## 🏗️ System Architecture

### Multi-Agent Pipeline

The system uses a **divide-and-conquer** approach with specialized agents:

```
Query Input → Planner → Search Agents → Writer → Email Agent → Final Report
```

#### Agent Responsibilities

**🧠 Planner Agent** (`planner_agent.py`)
- **Purpose**: Strategic planning of research approach
- **Input**: User's research query
- **Output**: 5 targeted search terms with reasoning
- **Why 5 searches?**: Balances comprehensiveness with efficiency

**🔍 Search Agent** (`search_agent.py`)  
- **Purpose**: Web search execution and summarization
- **Input**: Individual search terms + reasoning
- **Output**: 2-3 paragraph summaries (max 300 words)
- **Key Feature**: Concurrent execution for speed

**✍️ Writer Agent** (`writer_agent.py`)
- **Purpose**: Synthesis and report generation
- **Input**: Original query + all search summaries
- **Output**: Structured report (1000+ words) with follow-up questions
- **Format**: Professional markdown with sections and analysis

**📧 Email Agent** (`email_agent.py`)
- **Purpose**: Report formatting and delivery
- **Input**: Markdown report
- **Output**: HTML-formatted email via SendGrid
- **Features**: Professional styling and automatic sending

### Orchestration Layer

**🎼 Research Manager** (`research_manager.py`)
- Coordinates all agents in sequence
- Provides real-time status updates
- Handles error recovery and graceful failures
- Manages async operations and concurrency

**🖥️ User Interface** (`deep_research.py`)
- Clean Gradio web interface
- Real-time progress streaming
- Simple query input and report display

## 🚀 Getting Started

### Prerequisites

```bash
# Required Python packages
pip install gradio python-dotenv sendgrid asyncio pydantic

# AI agent framework (replace with your preferred framework)
pip install agents  # This appears to be a custom/proprietary framework
```

### Environment Setup

Create a `.env` file with your API keys:

```env
# SendGrid for email delivery
SENDGRID_API_KEY=your_sendgrid_api_key_here

# OpenAI API (for the agents framework)
OPENAI_API_KEY=your_openai_api_key_here
```

### Email Configuration

Update `email_agent.py` with your email addresses:

```python
from_email = Email("your_verified_sender@domain.com")
to_email = To("recipient@domain.com")
```

### Running the System

```bash
python deep_research.py
```

The Gradio interface will launch in your browser at `http://localhost:7860`

## 🔧 How It Works (Step by Step)

### 1. Query Planning Phase
```python
# User inputs: "Impact of AI on healthcare"
# Planner generates strategic searches like:
searches = [
    {"query": "AI healthcare applications 2024", "reason": "Current implementations"},
    {"query": "AI medical diagnosis accuracy", "reason": "Performance metrics"},
    {"query": "healthcare AI challenges risks", "reason": "Limitations and concerns"},
    {"query": "AI healthcare cost savings", "reason": "Economic impact"},
    {"query": "future AI healthcare trends", "reason": "Predictions and developments"}
]
```

### 2. Concurrent Search Execution
```python
# All searches run simultaneously using asyncio
tasks = [asyncio.create_task(self.search(item)) for item in search_plan.searches]
results = []
for task in asyncio.as_completed(tasks):
    result = await task
    if result is not None:  # Error handling
        results.append(result)
```

### 3. Intelligent Synthesis
The Writer Agent receives:
- Original query: "Impact of AI on healthcare"
- 5 search summaries covering different aspects
- Creates a structured report with sections like:
  - Executive Summary
  - Current Applications
  - Benefits and Challenges  
  - Future Outlook
  - Recommendations

### 4. Professional Delivery
- Converts markdown to HTML
- Applies professional styling
- Sends via SendGrid with appropriate subject line

## 🎓 Key Learning Concepts

### Async Programming Patterns
```python
# Concurrent execution pattern
async def perform_searches(self, search_plan: WebSearchPlan) -> list[str]:
    tasks = [asyncio.create_task(self.search(item)) for item in search_plan.searches]
    results = []
    for task in asyncio.as_completed(tasks):
        result = await task
        # Process results as they complete
```

### Agent Communication Patterns
```python
# Structured data passing between agents
class ReportData(BaseModel):
    short_summary: str
    markdown_report: str  
    follow_up_questions: list[str]
```

### Error Handling Strategy
```python
# Graceful degradation - continue with partial results
try:
    result = await Runner.run(search_agent, input)
    return str(result.final_output)
except Exception:
    return None  # Don't break the entire pipeline
```

### Streaming User Experience
```python
# Real-time progress updates
async def run(self, query: str):
    yield "Searches planned, starting to search..."
    # ... processing ...
    yield "Searches complete, writing report..."
    # ... more processing ...
    yield report.markdown_report  # Final result
```

## 🛠️ Customization Guide

### Adjusting Search Strategy

**Change number of searches:**
```python
# In planner_agent.py
HOW_MANY_SEARCHES = 8  # Increase for more comprehensive research
```

**Modify search agent instructions:**
```python
# In search_agent.py - adjust summary length/style
INSTRUCTIONS = (
    "You are a research assistant. Produce a detailed summary of 400-500 words..."  # More detailed
)
```

### Report Customization

**Adjust report length:**
```python
# In writer_agent.py
INSTRUCTIONS = (
    "Aim for 10-15 pages of content, at least 2000 words."  # Longer reports
)
```

**Add custom sections:**
```python
class ReportData(BaseModel):
    short_summary: str
    markdown_report: str
    follow_up_questions: list[str]
    executive_summary: str = Field(description="Executive summary section")  # New field
    methodology: str = Field(description="Research methodology used")  # New field
```

### Integration Options

**Database Storage:**
```python
# Add to research_manager.py
import sqlite3

async def save_report(self, query: str, report: ReportData):
    # Save to database for future reference
    conn = sqlite3.connect('research_reports.db')
    # ... database logic
```

**API Endpoints:**
```python
# Alternative to Gradio - FastAPI endpoints
from fastapi import FastAPI

app = FastAPI()

@app.post("/research")
async def research_endpoint(query: str):
    manager = ResearchManager()
    async for result in manager.run(query):
        # Stream results or return final report
        pass
```

## 🔍 Debugging and Monitoring

### Tracing Integration
The system includes OpenAI tracing for debugging:
```python
trace_id = gen_trace_id()
with trace("Research trace", trace_id=trace_id):
    # All operations are traced
    print(f"View trace: https://platform.openai.com/traces/trace?trace_id={trace_id}")
```

### Progress Monitoring
```python
# Real-time progress tracking
print(f"Searching... {num_completed}/{len(tasks)} completed")
```

### Error Logging
Consider adding comprehensive logging:
```python
import logging

logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)

# In search method
except Exception as e:
    logger.error(f"Search failed for {item.query}: {str(e)}")
    return None
```

## 🎯 Use Cases

### Academic Research
- Literature reviews
- Market research
- Technology assessments
- Policy analysis

### Business Intelligence  
- Competitive analysis
- Market trends
- Industry reports
- Due diligence

### Content Creation
- Blog post research
- Article fact-checking
- Background research
- Topic exploration

## 🔮 Advanced Extensions

### Multi-Modal Research
- Image search integration
- PDF document analysis
- Video content summarization

### Collaborative Features
- Team research sharing
- Report commenting
- Version control for reports

### Enhanced AI Capabilities
- Custom model fine-tuning
- Domain-specific agents
- Multi-language support

## 📝 Best Practices

### Query Formulation
- Be specific but not overly narrow
- Include context when helpful
- Consider multiple perspectives

**Good queries:**
- "Impact of remote work on software development productivity 2020-2024"
- "Sustainability practices in fast fashion industry"
- "Machine learning applications in financial fraud detection"

**Less effective queries:**
- "AI" (too broad)
- "Python coding" (too narrow/technical)

### System Maintenance
- Monitor API usage and costs
- Update agent instructions based on results
- Regular testing with diverse queries
- Email deliverability monitoring

## 🤝 Contributing

This system is modular and extensible. Consider contributing:

1. **New Agent Types**: Specialized agents for specific domains
2. **Output Formats**: PDF, presentations, infographics
3. **Data Sources**: Academic databases, news APIs, social media
4. **UI Improvements**: Better progress visualization, report sharing

## 📄 License

[Include your license information here]

## 🙏 Acknowledgments

Built using:
- [Gradio](https://gradio.app/) for the web interface
- [SendGrid](https://sendgrid.com/) for email delivery
- [Pydantic](https://pydantic.dev/) for data modeling
- OpenAI's GPT models for AI agents

---

**Ready to start researching?** Launch the system and try a query like "Future of renewable energy storage" to see the full pipeline in action!
