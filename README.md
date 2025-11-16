# Hierarchical Agents and Multimodal Output

This lesson culminates the course by building a sophisticated, multi-agent system that produces multimodal output (text and audio). It introduces hierarchical agent design, structured data schemas with Pydantic, and text-to-speech (TTS) generation.

## Key Concepts

- **Pydantic Schemas for Structured Output**: Instead of relying on prompts to format Markdown, this lesson uses Pydantic models (`NewsStory`, `AINewsReport`) to define a strict, typed schema for the agent's output. This ensures the data is structured, validated, and easier to work with programmatically.

- **Multimodal Output (Text-to-Speech)**: A new tool, `generate_podcast_audio`, is introduced. This tool leverages the Gemini API's TTS capabilities (`gemini-2.5-flash-preview-tts`) to convert a text script into a multi-speaker WAV audio file.

- **Hierarchical Agent Design**: This lesson implements a two-agent system:
    1.  **`root_agent` (AI News Podcast Producer)**: The main coordinator that orchestrates the entire workflow, from research to content formatting.
    2.  **`podcaster_agent` (Audio Generation Specialist)**: A specialized, single-purpose agent whose only job is to take a script and generate audio using the `generate_podcast_audio` tool. The `root_agent` delegates the audio generation task to this agent using an `AgentTool`.

- **Advanced Callback Logic**: The callback guardrails from Lesson 5 are refined:
    - `filter_news_sources_callback` is changed from a blocklist to a **whitelist**, ensuring that the agent *only* sources news from a pre-approved list of high-quality domains (`WHITELIST_DOMAINS`).
    - `enforce_data_freshness_callback` is added to automatically append a time filter to search queries, ensuring the news is recent.

## Code Breakdown

### Pydantic Models

- **`NewsStory`**: Defines the structure for a single news item, including fields like `company`, `ticker`, `summary`, `why_it_matters`, `financial_context`, and `source_domain`.
- **`AINewsReport`**: Defines the top-level structure for the entire report, containing a `title`, a `report_summary`, and a list of `NewsStory` objects.

### New and Updated Tools

- **`generate_podcast_audio`**: An `async` function that takes a script, calls the Gemini TTS API to generate audio with two distinct voices ('Joe' and 'Jane'), and saves the output as a `.wav` file.
- **`get_financial_context`**: Updated to be more resilient by filtering out invalid tickers like 'N/A' before making an API call.
- **`AgentTool(agent=podcaster_agent)`**: The `root_agent`'s toolset now includes a tool that is, in itself, another agent. This is the core of the hierarchical design.

### Agent Instructions and Workflow

The `root_agent`'s instructions are the most complex yet. It is tasked with a complete, end-to-end production workflow:

1.  Acknowledge the user's request.
2.  Silently perform research using the whitelisted and time-filtered `google_search`.
3.  Structure the findings using the `AINewsReport` Pydantic schema.
4.  Format the structured data into a Markdown report and save it.
5.  Convert the structured data into a conversational podcast script between 'Joe' and 'Jane'.
6.  Delegate the script to the `podcaster_agent` to generate the audio file.
7.  Provide a final confirmation to the user that both the report and the audio file are ready.

### Running the Agent

The notebook allows you to run this multi-agent system. After the agent completes its work, the notebook includes cells to both display the final Markdown report and play the generated `ai_today_podcast.wav` audio file, showcasing the full multimodal output.
