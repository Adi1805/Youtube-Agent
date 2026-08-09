# 🎥 YouTube AI Agent with LangChain & Gemini

> **Turn a YouTube link into information — not just a video.** 🤖🎥

An experimental **AI-powered YouTube Agent** built with **LangChain, Google Gemini, Python and YouTube utilities** that can understand a user's request, decide which tools are required, interact with those tools, and finally return a natural-language response.

Instead of building a separate script for every YouTube task, this project gives the language model access to a small toolkit and lets it decide what needs to happen.

For example, a user can simply ask:

```text
"Summarize this YouTube video in English:
https://youtu.be/xxxxxxxxxxx"
```

The agent can reason through the workflow:

```text
YouTube URL
     ↓
Extract Video ID
     ↓
Fetch Transcript
     ↓
Give Transcript to Gemini
     ↓
Generate Summary
```

But the same agent can also handle requests such as:

```text
"Search YouTube for the top 3 videos about X
and return their metadata and thumbnails."
```

In that case, the workflow changes:

```text
User Query
    ↓
Gemini
    ↓
Search YouTube
    ↓
Retrieve Video Results
    ↓
Get Metadata / Thumbnails
    ↓
Gemini
    ↓
Final Response
```

This project was built to understand one of the most important ideas behind modern LLM applications:

> **An LLM becomes much more useful when it can use tools instead of relying only on the knowledge contained inside its model.**

---

# 🚀 Project Overview

Most basic LLM applications follow a simple pattern:

```text
User → LLM → Answer
```

This project explores a more capable architecture:

```text
                 ┌───────────────────┐
                 │       User        │
                 └─────────┬─────────┘
                           │
                           ▼
                 ┌───────────────────┐
                 │   Gemini LLM      │
                 │  + Tool Calling   │
                 └─────────┬─────────┘
                           │
             ┌─────────────┼─────────────┐
             │             │             │
             ▼             ▼             ▼
      Extract Video   Fetch Transcript  Search YouTube
           ID               │             │
             │              │             │
             ▼              ▼             ▼
       Video ID        Transcript      Video Results
                           │             │
                           └──────┬──────┘
                                  │
                                  ▼
                         Metadata / Thumbnails
                                  │
                                  ▼
                           Gemini Response
                                  │
                                  ▼
                            Final Answer
```

The model is not directly executing Python functions.

Instead, it identifies the tool it needs and generates structured tool-call arguments. The Python application executes that tool, returns the result to the model, and the model continues from there.

This is the core **tool-calling / agentic workflow** explored in this project.

---

# 🎯 Objectives

The main goals of this project were to understand and implement:

- 🔧 LLM tool calling
- 🤖 Agent-style workflows
- 🦜 LangChain tool integration
- 💎 Google Gemini integration
- 🎥 YouTube search
- 📝 YouTube transcript extraction
- 🔎 YouTube URL parsing
- 📊 YouTube metadata extraction
- 🖼️ Thumbnail retrieval
- 🔄 Multi-step tool execution
- 🧠 LLM-driven tool selection
- 🔁 Recursive tool-call processing
- ⛓️ LangChain Runnable pipelines

Rather than treating Gemini as a simple chatbot, the project explores how it can act as a **decision-making layer over external Python tools**.

---

# ✨ Features

## 🎥 1. YouTube Video ID Extraction

The agent includes a dedicated tool for extracting the 11-character YouTube video ID from URLs.

It supports common URL patterns such as:

```text
https://www.youtube.com/watch?v=VIDEO_ID
https://youtu.be/VIDEO_ID
https://www.youtube.com/embed/VIDEO_ID
```

The tool uses a regular expression to identify the video ID and returns an error when the URL cannot be parsed.

Implemented as:

```python
@tool
def extract_video_id(url: str) -> str:
    ...
```

---

# 📝 2. YouTube Transcript Extraction

Once the video ID has been identified, the agent can request the video's transcript.

The project uses:

```python
YouTubeTranscriptApi
```

and exposes transcript retrieval as a LangChain tool:

```python
@tool
def fetch_transcript(
    video_id: str,
    language: str = "en"
) -> str:
    ...
```

The transcript snippets are joined together into a single text string before being returned to the language model.

This creates the bridge between:

```text
YouTube Video
      ↓
Transcript
      ↓
LLM
      ↓
Summary
```

The notebook demonstrates this workflow using an actual YouTube video.

---

# 🔎 3. YouTube Search

The agent can also search YouTube based on natural-language queries.

For example:

```text
Search YouTube for:
"Top 3 videos of Commonwealth Games 2026 for Team India"
```

The search tool returns information such as:

```json
{
    "title": "Video Title",
    "video_id": "abc123",
    "url": "https://youtu.be/abc123"
}
```

This means the LLM can move from:

```text
Natural Language Query
        ↓
YouTube Search
        ↓
Structured Video Results
```

instead of requiring the user to manually find video links.

---

# 📊 4. YouTube Metadata Extraction

The project also provides a tool for retrieving richer information about a YouTube video.

The metadata function extracts fields including:

- Video title
- Number of views
- Duration
- Channel / uploader
- Likes
- Comments
- Chapters

This is handled using:

```python
yt_dlp
```

through the custom:

```python
get_full_metadata()
```

tool.

The result is returned as a Python dictionary that can then be passed back into the agent workflow.

---

# 🖼️ 5. Thumbnail Extraction

The project also includes a thumbnail retrieval tool.

For a given YouTube URL, it extracts the available thumbnail URLs along with information such as:

```text
Width
Height
Resolution
URL
```

This makes the agent capable of responding to requests that go beyond textual information.

For example:

```text
"Show me the top videos about X
along with their metadata and thumbnails."
```

The agent can combine multiple tools to fulfil the request.

---

# 🤖 6. Gemini + LangChain Tool Calling

The central component of the project is the integration between:

```text
Google Gemini
        +
LangChain
        +
Custom Python Tools
```

The notebook initializes Gemini through:

```python
ChatGoogleGenerativeAI(
    model="gemini-flash-latest",
    temperature=0
)
```

and then binds the custom tools:

```python
llm_with_tools = llm.bind_tools(tools)
```

The available tools include:

```text
extract_video_id
fetch_transcript
search_youtube
get_full_metadata
get_thumbnails
```

The model can therefore determine which function is appropriate for a given request.

---

# 🧠 How the Agent Thinks About a Request

Consider:

```text
"Summarize this YouTube video:
https://youtu.be/XXXXXXXXXXX"
```

The LLM does not need to be explicitly told:

```text
1. Extract the ID
2. Fetch transcript
3. Summarize transcript
```

Instead, it can identify the appropriate tool calls.

Conceptually:

```text
                     User
                      │
                      ▼
              ┌──────────────┐
              │    Gemini    │
              └──────┬───────┘
                     │
              Tool required?
                     │
                    YES
                     │
                     ▼
          extract_video_id()
                     │
                     ▼
                Video ID
                     │
                     ▼
              ┌──────────────┐
              │    Gemini    │
              └──────┬───────┘
                     │
                     ▼
          fetch_transcript()
                     │
                     ▼
                Transcript
                     │
                     ▼
              ┌──────────────┐
              │    Gemini    │
              └──────┬───────┘
                     │
                     ▼
               Final Summary
```

This is the fundamental agent loop explored in the project.

---

# 🔄 Manual Tool-Calling Workflow

One of the most useful parts of the notebook is that the tool-calling process is implemented manually before being abstracted into a reusable agent.

The workflow explicitly:

1. Sends the user query to Gemini.
2. Reads the returned tool call.
3. Identifies the tool name.
4. Extracts the tool arguments.
5. Executes the corresponding Python function.
6. Creates a `ToolMessage`.
7. Sends the result back to Gemini.
8. Checks whether another tool is required.
9. Produces the final response.

The notebook maintains a mapping:

```python
tool_mapping = {
    "get_thumbnails": get_thumbnails,
    "extract_video_id": extract_video_id,
    "fetch_transcript": fetch_transcript,
    "search_youtube": search_youtube,
    "get_full_metadata": get_full_metadata
}
```

This makes the relationship between the model's requested function and the actual Python implementation explicit.

---

# 🧩 Generic Tool Executor

The project later abstracts tool execution into:

```python
def execute_tool(tool_call):
    ...
```

Instead of writing separate execution logic for every function, the executor:

```text
Tool Call
    ↓
Identify Tool
    ↓
Extract Arguments
    ↓
Execute Tool
    ↓
Convert Result to ToolMessage
    ↓
Return to LLM
```

This is an important step toward making the system reusable.

Adding a new tool does not require completely rewriting the agent loop.

---

# 🔁 Multi-Step Agent Execution

The project implements:

```python
def run_agent(query, max_steps=8):
    ...
```

This function allows the LLM to repeatedly call tools until it has enough information to answer the user.

The high-level loop is:

```text
User Query
     ↓
Gemini
     ↓
Tool Call?
  ↙       ↘
 YES       NO
  ↓         ↓
Execute    Final
Tool       Answer
  ↓
Tool Result
  ↓
Gemini
  ↓
Tool Call?
  ↓
 ...
```

A maximum number of steps is included to prevent an uncontrolled loop.

---

# ⛓️ LangChain Runnable Pipeline

After implementing the manual workflow, the project explores LangChain's Runnable abstraction.

The notebook creates individual processing stages:

```python
initial_setup
first_llm_call
first_tool_processing
second_llm_call
second_tool_processing
final_summary
```

These are then composed into:

```python
chain = (
    initial_setup
    | first_llm_call
    | first_tool_processing
    | second_llm_call
    | second_tool_processing
    | final_summary
)
```

This provides a more structured way to think about the agent as a sequence of transformations.

---

# ♾️ Recursive Tool-Calling Agent

The most interesting evolution in the notebook is the recursive approach.

Instead of assuming that the agent will always need exactly one or two tool calls, the project introduces:

```python
def should_continue(messages):
    ...
```

and:

```python
def _recursive_chain(messages):
    ...
```

The idea is simple:

```text
LLM Response
     │
     ▼
Does it contain tool calls?
     │
 ┌───┴────┐
 │        │
YES       NO
 │        │
 ▼        ▼
Execute  Finish
Tools
 │
 ▼
Call LLM Again
 │
 ▼
Check Again
 │
 └───────────────┐
                 │
                 ▼
              Repeat
```

This makes the workflow much more flexible.

The agent is no longer limited to a hard-coded number of tool-call rounds.

---

# 🏗️ System Architecture

```text
                          ┌─────────────────────┐
                          │        USER         │
                          └──────────┬──────────┘
                                     │
                              Natural Language
                                     │
                                     ▼
                          ┌─────────────────────┐
                          │   Google Gemini     │
                          │     LLM Layer       │
                          └──────────┬──────────┘
                                     │
                              Tool Decision
                                     │
                    ┌────────────────┼────────────────┐
                    │                │                │
                    ▼                ▼                ▼
             ┌────────────┐  ┌──────────────┐ ┌──────────────┐
             │ Extract ID │  │  Transcript  │ │ YouTube      │
             │    Tool    │  │     Tool     │ │ Search Tool  │
             └────────────┘  └──────────────┘ └──────────────┘
                    │                │                │
                    └────────────────┼────────────────┘
                                     │
                    ┌────────────────┴────────────────┐
                    │                                 │
                    ▼                                 ▼
             ┌──────────────┐                 ┌──────────────┐
             │   Metadata   │                 │  Thumbnail   │
             │     Tool     │                 │     Tool     │
             └──────────────┘                 └──────────────┘
                    │                                 │
                    └────────────────┬────────────────┘
                                     │
                                Tool Results
                                     │
                                     ▼
                          ┌─────────────────────┐
                          │   Gemini + Context  │
                          └──────────┬──────────┘
                                     │
                                     ▼
                          ┌─────────────────────┐
                          │    Final Answer     │
                          └─────────────────────┘
```

---

# 🔧 Tools Available to the Agent

| Tool | Purpose | Input | Output |
|---|---|---|---|
| `extract_video_id` | Extract YouTube video ID | YouTube URL | Video ID |
| `fetch_transcript` | Retrieve transcript | Video ID + language | Transcript |
| `search_youtube` | Search YouTube | Search query | Video results |
| `get_full_metadata` | Retrieve video information | YouTube URL | Metadata dictionary |
| `get_thumbnails` | Retrieve thumbnails | YouTube URL | Thumbnail information |

These tools form the agent's external capabilities.

The LLM itself does not directly access YouTube through these functions. It requests the tool call, while the Python application executes the function and returns the result.

This separation is a key part of the architecture.

---

# 🛠️ Technology Stack

| Technology | Role |
|---|---|
| 🐍 Python | Core programming language |
| 🦜 LangChain | Tool orchestration and LLM workflow |
| 💎 Google Gemini | Large Language Model |
| 🎥 YouTube | Video source |
| 📝 YouTube Transcript API | Transcript extraction |
| 📥 yt-dlp | Video metadata and thumbnail extraction |
| 🔎 Pytube | YouTube search and URL utilities |
| 📓 Jupyter / Google Colab | Development environment |

The notebook installs and uses `pytube`, `youtube-transcript-api`, `langchain-community`, `langchain-openai`, `yt-dlp`, `google-genai`, `langchain-google-genai` and LangChain itself.

---

# 📂 Project Structure

A clean GitHub repository can be organized as:

```text
YouTube-Agent-with-LangChain/
│
├── 📓 Youtube_Agent_with_Langchain.ipynb
├── 📄 README.md
├── 📄 requirements.txt
├── 📄 .env.example
├── 🚫 .gitignore
│
└── 📁 assets/
    └── screenshots/
```

---

# ⚙️ Installation

## 1. Clone the Repository

```bash
git clone https://github.com/your-username/youtube-agent-with-langchain.git

cd youtube-agent-with-langchain
```

---

## 2. Create a Virtual Environment

### Windows

```bash
python -m venv venv

venv\Scripts\activate
```

### Linux / macOS

```bash
python3 -m venv venv

source venv/bin/activate
```

---

# 📦 3. Install Dependencies

```bash
pip install -U langchain
pip install -U langchain-community
pip install -U langchain-google-genai
pip install -U google-genai
pip install -U pytube
pip install -U youtube-transcript-api
pip install -U yt-dlp
```

You can also create a `requirements.txt` file containing:

```text
langchain
langchain-community
langchain-google-genai
google-genai
pytube
youtube-transcript-api
yt-dlp
```

---

# 🔐 4. Configure the Gemini API Key

**Never hard-code your API key inside the notebook or commit it to GitHub.**

Create a `.env` file:

```text
GOOGLE_API_KEY=your_google_api_key_here
```

Then load it through your application environment.

For example:

```python
import os

from dotenv import load_dotenv

load_dotenv()

google_api_key = os.getenv("GOOGLE_API_KEY")
```

Add `.env` to `.gitignore`:

```text
.env
```

### ⚠️ Security Warning

The original uploaded notebook contains a Google API key directly in the code.

Before publishing this project:

1. **Revoke or rotate that key.**
2. Remove the exposed key from the notebook.
3. Create a new key if required.
4. Store it through environment variables.
5. Never commit secrets to GitHub.

This is especially important because a GitHub repository is publicly searchable once published.

---

# ▶️ Running the Project

Open the notebook:

```text
Youtube_Agent_with_Langchain.ipynb
```

Then execute the cells sequentially.

The notebook demonstrates several types of queries.

### Video summarization

```text
Summarize this YouTube video:
https://youtu.be/VIDEO_ID
```

### YouTube search

```text
Search YouTube for videos about
Artificial Intelligence
```

### Search + metadata

```text
Search YouTube for the top 3 videos about
a particular topic and return their metadata.
```

### Search + metadata + thumbnails

```text
Show the top 3 YouTube videos about a topic
with metadata and thumbnails.
```

---

# 🧪 Example Workflow 1 — Video Summarization

Input:

```text
Summarize this YouTube video:
https://youtu.be/VIDEO_ID
```

The agent can perform:

```text
User
 ↓
Gemini
 ↓
extract_video_id
 ↓
Gemini
 ↓
fetch_transcript
 ↓
Gemini
 ↓
Summary
```

The final result is generated from the retrieved transcript rather than requiring the model to directly watch the video.

---

# 🧪 Example Workflow 2 — YouTube Search

Input:

```text
Search YouTube for videos about
Artificial Intelligence.
```

Workflow:

```text
User
 ↓
Gemini
 ↓
search_youtube()
 ↓
Video Results
 ↓
Gemini
 ↓
Human-readable response
```

---

# 🧪 Example Workflow 3 — Search + Metadata

Input:

```text
Search YouTube for the top 3 videos
about Commonwealth Games 2026 for Team India
and return their metadata.
```

The agent can combine:

```text
search_youtube()
       ↓
Video URLs
       ↓
get_full_metadata()
       ↓
Metadata
       ↓
Gemini
       ↓
Final Response
```

This demonstrates why tool calling becomes useful when a single request requires multiple external operations.

---

# 🧪 Example Workflow 4 — Metadata + Thumbnails

The notebook also explores requests such as:

```text
Show top 3 CJP protest videos in India
with metadata and thumbnails.
```

This requires the agent to combine multiple capabilities rather than relying on a single function.

Conceptually:

```text
Search
  ↓
Video IDs / URLs
  ↓
┌───────────────┬─────────────────┐
│               │                 │
▼               ▼                 ▼
Metadata     Thumbnails       Other Tools
│               │
└───────────────┘
        ↓
   Gemini Context
        ↓
   Final Response
```

The notebook contains this type of multi-tool query in its final recursive-agent experiment.

---

# 🧠 Key Concepts Learned

## 1. Tool Calling

The model decides:

```text
"What function should I call?"
```

The application decides:

```text
"How should that function actually execute?"
```

This distinction is extremely important.

---

## 2. Agents

An agent can dynamically decide what actions are necessary to answer a request.

Instead of:

```text
Fixed Pipeline
```

we move toward:

```text
Goal
 ↓
Decision
 ↓
Action
 ↓
Observation
 ↓
Decision
 ↓
Action
 ↓
...
```

---

## 3. Tool Messages

Tool outputs are inserted back into the conversation as structured messages.

The notebook uses:

```python
ToolMessage
```

to connect tool execution results with the original tool call.

---

## 4. Runnable Pipelines

LangChain's Runnable abstractions allow individual processing stages to be composed into a larger workflow.

This makes the system easier to reason about and extend.

---

## 5. Recursive Workflows

A fixed number of tool calls may work for simple tasks.

But real-world agentic applications often need a variable number of steps.

The recursive implementation in this project explores exactly that idea.

---

# 🔄 Evolution of the Project

One of the things I found most useful while building this project was seeing the architecture evolve.

### Stage 1 — Individual Tools

First, each capability was implemented separately:

```text
Extract ID
Fetch Transcript
Search YouTube
Get Metadata
Get Thumbnails
```

### Stage 2 — Bind Tools to Gemini

```text
Tools
  ↓
Gemini
```

### Stage 3 — Manual Tool Calling

```text
Gemini
 ↓
Tool Call
 ↓
Execute
 ↓
Tool Result
 ↓
Gemini
```

### Stage 4 — Reusable Agent Function

```text
run_agent()
```

### Stage 5 — Runnable Pipeline

```text
RunnablePassthrough
       ↓
LLM
       ↓
Tool Processing
       ↓
LLM
       ↓
Final Summary
```

### Stage 6 — Recursive Agent

```text
LLM
 ↓
Tool?
 ↓
YES → Execute → LLM → Tool?
                    ↓
                   YES
                    ↓
                  ...
                    ↓
                   NO
                    ↓
                 Answer
```

This progression is a major part of what makes the project useful as a learning exercise.

---

# 📈 Possible Improvements

This project is intentionally experimental, but there are several ways it could be taken further.

## 🔹 1. Add a Web Interface

A frontend could be built using:

- Streamlit
- Gradio
- React
- FastAPI + frontend

The user could simply paste:

```text
YouTube URL
```

and receive:

```text
Title
Transcript
Summary
Key Points
Metadata
Thumbnail
```

---

## 🔹 2. Add Timestamped Summaries

Instead of returning only a single summary:

```text
00:00 Introduction
02:15 Main Topic
07:42 Important Example
12:30 Conclusion
```

The transcript timestamps could be preserved and used to create chapter-wise summaries.

---

## 🔹 3. Add Question Answering

The system could become a conversational YouTube assistant:

```text
User:
What was discussed in the video?

Agent:
...

User:
What did the speaker say about AI?

Agent:
...

User:
At what point was that discussed?

Agent:
...
```

---

## 🔹 4. Add Multiple Language Support

The transcript tool already accepts a language parameter.

This could be extended into:

```text
English
Hindi
Marathi
Spanish
French
German
...
```

The agent could then retrieve or process transcripts according to the user's language.

---

## 🔹 5. Add Structured Output

Instead of returning free-form text, the agent could produce:

```json
{
  "title": "...",
  "channel": "...",
  "duration": "...",
  "summary": "...",
  "key_points": [],
  "chapters": []
}
```

This would make the output easier to consume in applications.

---

## 🔹 6. Add Persistent Memory

A future version could remember:

```text
Previously summarized videos
User interests
Previously asked questions
Saved videos
```

This would turn the project into a more complete personal YouTube research assistant.

---

## 🔹 7. Add RAG

Transcripts could be stored inside a vector database.

Then the architecture becomes:

```text
YouTube
   ↓
Transcript
   ↓
Chunking
   ↓
Embeddings
   ↓
Vector Database
   ↓
Retriever
   ↓
Gemini
   ↓
Answer
```

This would allow users to ask detailed questions about long videos without repeatedly passing the entire transcript to the LLM.

---

# ⚠️ Limitations

The current project has some practical limitations.

### Transcript availability

Not every YouTube video will have an accessible transcript in the requested language.

### External dependency

The project depends on external YouTube-related services and libraries. Their behaviour can change over time.

### Tool failures

YouTube search, metadata extraction or transcript retrieval can fail because of:

- unavailable videos
- network problems
- missing transcripts
- changes to YouTube
- library compatibility issues
- rate limits

### API costs

Gemini API usage may incur costs depending on the selected model, account and usage.

### LLM decisions are not guaranteed

The model may sometimes select an unnecessary tool, provide incorrect arguments or require additional prompting.

This is one reason the project includes explicit tool execution, error handling and maximum-step controls.

---

# 🔒 Security Considerations

This project interacts with external APIs and therefore should follow basic secret-management practices.

### Never do this:

```python
GOOGLE_API_KEY = "actual-secret-key"
```

### Prefer:

```python
GOOGLE_API_KEY = os.getenv("GOOGLE_API_KEY")
```

and keep secrets in:

```text
.env
```

with:

```text
.env
```

added to:

```text
.gitignore
```

Also avoid committing:

- API keys
- access tokens
- cookies
- private YouTube credentials
- personal account information

---

# 📚 References

The references below are intentionally restricted to **2022–2026**, as requested.

### 1. Google Gemini API — Function Calling

Google's Gemini documentation describes function calling as a mechanism through which the model can select external functions, provide structured arguments, and receive the function result back before generating a final response. This is directly relevant to the architecture used in this project.

**Google AI for Developers — Function Calling with Gemini API, 2026**

[Gemini API — Function Calling](https://ai.google.dev/gemini-api/docs/function-calling?hl=en&utm_source=chatgpt.com)

---

### 2. Google Gemini API — Tools

The Gemini documentation explains how tool calls are returned to an application, how the application executes the function, and how the result is returned to the model.

**Google AI for Developers — Using Tools with Gemini API, 2026**

[Gemini API — Using Tools](https://ai.google.dev/gemini-api/docs/tools?authuser=1&utm_source=chatgpt.com)

---

### 3. LangChain — Tool Calling

LangChain's current documentation describes tool calls containing a tool name, structured arguments and an identifier connecting the call to its result. This closely matches the `tool_calls`, `ToolMessage` and tool-mapping approach used in this project.

**LangChain Documentation — Tool Calling, 2026**

[LangChain — Tool Calling Documentation](https://docs.langchain.com/oss/python/langchain/frontend/tool-calling?utm_source=chatgpt.com)

---

### 4. LangChain Releases

The LangChain project continues to evolve its agent and tool-calling ecosystem. The project's release history provides a useful reference for version changes and ongoing development.

**LangChain — GitHub Releases, 2026**

[LangChain GitHub Releases](https://github.com/langchain-ai/langchain/releases?utm_source=chatgpt.com)

---

### 5. YouTube Data API Documentation

Google's YouTube Data API documentation describes the official mechanisms for searching YouTube resources and retrieving video-related information.

**Google for Developers — YouTube Data API Reference, 2026**

[YouTube Data API Reference](https://developers.google.com/youtube/v3/docs?utm_source=chatgpt.com)

---

### 6. YouTube Data API — Search

The YouTube Data API `search.list` endpoint supports searching for videos, channels and playlists and provides fields such as video IDs, titles, descriptions and thumbnails.

**Google for Developers — YouTube Data API Search, 2026**

[YouTube Data API Search Documentation](https://developers.google.com/youtube/v3/docs/search?hl=en&utm_source=chatgpt.com)

---

### 7. YouTube Transcript API

The `youtube-transcript-api` project provides a Python interface for retrieving YouTube transcripts and subtitles. Its recent releases include versions published in 2025 and 2026.

**YouTube Transcript API — PyPI, 2026**

[youtube-transcript-api on PyPI](https://pypi.org/project/youtube-transcript-api/1.2.2/?utm_source=chatgpt.com)

---

### 8. yt-dlp

`yt-dlp` is used in this project for extracting YouTube video information such as metadata and thumbnails. The project remains actively maintained, with releases continuing through 2026.

**yt-dlp — GitHub Repository, 2026**

[yt-dlp GitHub Repository](https://github.com/yt-dlp/yt-dlp?utm_source=chatgpt.com)

---

### 9. yt-dlp Documentation

The official yt-dlp documentation provides installation and usage information and reflects the continuing development of the project.

**yt-dlp Wiki — Installation, 2025**

[yt-dlp Installation Documentation](https://github.com/yt-dlp/yt-dlp/wiki/Installation?utm_source=chatgpt.com)

---

# 📌 What This Project Demonstrates

At a high level, this project demonstrates the transition from:

```text
Traditional LLM Application
```

to:

```text
Tool-Using LLM Application
```

and eventually toward:

```text
Agentic Application
```

The progression can be visualized as:

```text
                ┌─────────────────────┐
                │      Basic LLM      │
                │                     │
                │ User → Model → Text │
                └──────────┬──────────┘
                           │
                           ▼
                ┌─────────────────────┐
                │    Tool-Using LLM   │
                │                     │
                │ Model → Tool → Data │
                └──────────┬──────────┘
                           │
                           ▼
                ┌─────────────────────┐
                │       Agent         │
                │                     │
                │ Decide → Act →      │
                │ Observe → Decide... │
                └─────────────────────┘
```

And that is ultimately what this project is about.

Not just making a chatbot that can summarize a YouTube video.

But understanding how an LLM can **decide what information it needs, call external tools to obtain it, process the returned information, and continue until it can answer the user's request.**

---

# 💭 Final Thoughts

Building this project helped me understand something that is easy to miss when learning about LLMs:

**The model itself is only one part of an AI application.**

The real power starts appearing when the model can interact with the outside world.

In this project, Gemini can:

```text
Understand the request
        ↓
Choose a tool
        ↓
Call the tool
        ↓
Receive the result
        ↓
Decide what to do next
        ↓
Call another tool if necessary
        ↓
Generate the final answer
```

That shift from simply **generating text** to **taking actions through tools** is what makes agentic AI applications so interesting.

And this YouTube Agent is my exploration of that idea.

---


> **From a YouTube URL to an AI agent that can search, retrieve, reason and respond. 🚀**
