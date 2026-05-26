# LM Studio MCP Server Configuration

This document outlines the setup, requirements, and mechanics of the provided Model Context Protocol (MCP) configuration.

This configuration equips your local Large Language Models (LLMs) in LM Studio with the ability to search the live web and look up factual information on Wikipedia.

---

## 📌 What is this?

The provided JSON snippet is an MCP Server Configuration. The Model Context Protocol (MCP) is an open standard that allows AI models to securely connect to external tools and data sources.

By adding this JSON to your LM Studio setup, you are instructing LM Studio to spin up two background applications (servers) that your local AI models can talk to:

* **DuckDuckGo Search:** Gives the model access to live web search.
* **Wikipedia:** Gives the model access to search and read Wikipedia articles.

---

## ⚙️ Prerequisites: You Must Install uv

If you look closely at the configuration, both servers use the command "uvx".

uvx is a command-line tool powered by uv, an extremely fast Python package manager built in Rust. It allows you to run Python tools in isolated environments without having to manually set up virtual environments or install packages globally.

For this configuration to work, you must install uv on your system first.

### How to install uv

#### macOS / Linux

Open your terminal and run:

```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
```

#### Windows

Open PowerShell and run:

```powershell
powershell -ExecutionPolicy ByPass -c "irm https://astral.sh/uv/install.ps1 | iex"
```

(You can verify the installation by typing `uv --version` or `uvx --version` in your terminal after restarting it).

---

## 🛠️ What These Servers Do

Here is a breakdown of the two servers defined in the JSON:

### 1. ddg-search (DuckDuckGo Search)

Command executed: uvx duckduckgo-mcp-server

* **What it does:** It acts as a bridge between the AI model and the DuckDuckGo search engine. If you ask the AI a question about current events, it can use this tool to query the web and read the search results.
* **Why it's useful:** It grounds your local, offline AI in real-time reality, allowing it to answer questions about things that happened after its training data cutoff.

### 2. wikipedia

Command executed: uvx --from mcp-server-wikipedia wikipedia-mcp-server

* **What it does:** It allows the AI model to search for specific Wikipedia articles and read their contents.
* **Why it's useful:** It provides a highly reliable, factual data source for the AI, reducing hallucinations when asking for historical facts, biographies, or scientific summaries.

---

## 🧠 How it Works in LM Studio

Here is the step-by-step process of how this configuration operates under the hood in LM Studio:

1. **Initialisation:** When you load LM Studio with this configuration, LM Studio detects the `mcpServers` block. It uses your system's `uvx` command to download and start both the DuckDuckGo and Wikipedia MCP servers in the background.
2. **Tool Registration:** The MCP servers send a list of their capabilities (tools) to LM Studio. LM Studio tells the loaded LLM, "Hey, you now have tools available to search the web and read Wikipedia."
3. **The Prompt:** You ask the model a question (e.g., "What is the latest news regarding the Mars Rover?").
4. **The Tool Call:** The LLM realises it doesn't know the answer from its internal memory. Instead of generating text immediately, it outputs a special "tool call" requesting to use the `ddg-search` tool.
5. **The Execution:** LM Studio intercepts this tool call, pauses the AI, and forwards the search query to the `ddg-search` MCP server.
6. **The Context Injection:** The MCP server fetches the results from DuckDuckGo and hands them back to LM Studio. LM Studio injects those search results invisibly into the AI's context window.
7. **The Final Answer:** The AI reads the newly provided search results and generates a natural language response to your original question.

---

## 💻 The MCP Server Configuration

Inside the LM studio app, you will see the Developer tab on the left side. Click this, and then at the top of the page, you will see mcp.json. Click there, remove what is there and paste this in, and you will have them in the tools icon in your input bar.

Here is the configuration block, ready to copy and paste:

```json
{
  "mcpServers": {
    "ddg-search": {
      "command": "uvx",
      "args": [
        "duckduckgo-mcp-server"
      ]
    },
    "wikipedia": {
      "command": "uvx",
      "args": [
        "--from",
        "mcp-server-wikipedia",
        "wikipedia-mcp-server"
      ]
    }
  }
}
```
