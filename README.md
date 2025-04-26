# Google ADK Agent Setup

This document outlines the setup process and benefits observed from the provided Google ADK agent codebase, which includes a Coordinator agent managing three sub-agents: Async Reddit Scout, Summarizer, and Speaker.

## Setup Process

1.  **Project Structure:** The project is organized into separate directories for each agent (`async_reddit_scout`, `coordinator`, `speaker`, `summarizer`), promoting modularity. Each directory contains an `__init__.py` and `agent.py`.

2.  **Dependencies:**
    *   **Python Packages:** Ensure Python is installed. Install necessary packages using pip:
        ```bash
        pip install google-adk python-dotenv lite_llm uvx elevenlabs-mcp git+https://github.com/adhikasp/mcp-reddit.git#egg=mcp-reddit
        # Or install from requirements.txt if provided
        # pip install -r requirements.txt
        ```
        *   `google-adk`: The core framework for building agents.
        *   `python-dotenv`: To load environment variables from a `.env` file.
        *   `lite_llm`: Used as a wrapper/interface for the underlying LLMs (Gemini models).
        *   `uvx`: A tool likely used for managing and running external Python applications or MCP servers in isolated environments. It's used here to run the `mcp-reddit` and `elevenlabs-mcp` servers.
        *   `elevenlabs-mcp`: The MCP server package for ElevenLabs TTS.
        *   `mcp-reddit`: The MCP server package for Reddit interaction (installed via git).
    *   **External Tools:** `uvx` needs to be installed and accessible in the system's PATH.

3.  **Environment Variables:**
    *   Create a `.env` file in the project root directory (alongside the agent directories).
    *   Add the following required API keys:
        ```dotenv
        GOOGLE_API_KEY=YOUR_GOOGLE_API_KEY_HERE
        ELEVENLABS_API_KEY=YOUR_ELEVENLABS_API_KEY_HERE
        ```
    *   The code uses `dotenv` to load these variables.

4.  **MCP Server Integration:**
    *   **Reddit Scout:** Connects to the `mcp-reddit` server using `uvx`. The `async_reddit_scout/agent.py` script attempts to start this server via `uvx --from git+https://github.com/adhikasp/mcp-reddit.git mcp-reddit`. It expects a tool named `fetch_reddit_hot_threads`.
    *   **Speaker:** Connects to the `elevenlabs-mcp` server using `uvx`. The `speaker/agent.py` script attempts to start this server via `uvx elevenlabs-mcp`, passing the `ELEVENLABS_API_KEY` as an environment variable to the MCP process. It expects a TTS tool (likely named `text_to_speech`).
    *   **Error Handling:** Both agents include basic error handling if `uvx` is not found or if the connection to the MCP server fails.

5.  **Agent Configuration:**
    *   **Models:** Different agents utilize specific Google Gemini models (`gemini-1.5-flash-latest`, `gemini-1.5-pro-latest`) via `LiteLlm`.
    *   **Instructions:** Each agent has specific instructions defining its role, how to use its tools (if any), and how to interact with other agents (in the case of the coordinator).
    *   **Tools:** Agents are configured with tools discovered from their respective MCP servers.
    *   **Sub-Agents:** The `coordinator` agent is configured with the other three agents (`reddit_agent`, `summarizer_agent`, `speaker_agent`) as sub-agents.

6.  **Asynchronous Handling:**
    *   The `async_reddit_scout` and `speaker` agents are asynchronous (`async def create_agent()`), likely due to the asynchronous nature of connecting to and interacting with MCP servers via `uvx`.
    *   The `coordinator` uses `AsyncExitStack` to manage the lifecycles (startup/shutdown) of the asynchronous sub-agents and their MCP connections.

7.  **Running the Agent:** The entry point would typically involve importing and running the `coordinator.root_agent`. (Specific run command/script not provided, but this is the standard ADK pattern).

## Benefits of this Architecture

*   **Modularity:** Each agent has a distinct responsibility (fetching, summarizing, speaking), making the codebase easier to understand, maintain, and test.
*   **Extensibility:** Leverages the Model Context Protocol (MCP) to integrate external services (Reddit, ElevenLabs TTS) via dedicated tools (`mcp-reddit`, `elevenlabs-mcp`). This allows adding new capabilities without modifying the core agent logic significantly, simply by adding new MCP tools/servers.
*   **Coordination & Delegation:** The `coordinator` agent acts as a central orchestrator, intelligently delegating tasks to the appropriate sub-agent based on the user's request. This follows the principle of separation of concerns.
*   **Abstraction:** The use of MCP tools abstracts the complexities of interacting with external APIs (Reddit API, ElevenLabs API). The agents only need to know the tool's interface (name, parameters).
*   **Asynchronous Operations:** Efficiently handles potentially long-running I/O operations (like network requests to MCP servers or external APIs) using `asyncio`, preventing the application from blocking. `AsyncExitStack` ensures proper resource management for these async components.
*   **Tool Management:** `MCPToolset.from_server` dynamically discovers tools provided by the MCP servers, making the agent adaptable if the server's tools change.
*   **Environment Isolation (via uvx):** Using `uvx` likely helps manage dependencies for the MCP servers separately from the main agent application, preventing conflicts.
*   **Clear Instructions:** Each agent has well-defined instructions, guiding the LLM on its specific role and how to interact with tools and other agents.



Run using - adk web

Developed using Python 3.10

LiteLLM is a  Python library and proxy server that simplifies the integration of various Large Language Model (LLM) APIs,allowing developers to interact with hundreds of LLMs using a consistent OpenAI-compatible API format

