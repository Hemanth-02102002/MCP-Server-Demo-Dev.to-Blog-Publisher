# 🚀 MCP Server Demo — Dev.to Blog Publisher

> A **Model Context Protocol (MCP)** server that empowers Claude AI to autonomously publish blog posts to [Dev.to](https://dev.to) — turning a simple conversation into a fully automated content publishing pipeline.

---

## 📖 Table of Contents

- [Project Description](#-project-description)
- [Overview](#-overview)
- [Tools & Technologies](#-tools--technologies)
- [Workflow Architecture](#-workflow-architecture)
- [Connecting with Claude Desktop](#-connecting-with-claude-desktop)
- [How to Run the Project](#-how-to-run-the-project)
- [Environment Variables](#-environment-variables)
- [Project Structure](#-project-structure)

---

## 📌 Project Description

**MCP Server Demo** is a custom MCP (Model Context Protocol) server built with Python. It exposes tools and prompt templates that allow a Claude AI client (such as **Claude Desktop**) to:

- **Generate** a complete, developer-focused blog post on any given topic.
- **Publish** that blog post directly to [Dev.to](https://dev.to) via its REST API — either as a **draft** or **live article**.

This project showcases how the MCP standard bridges the gap between a Large Language Model (LLM) and external services, enabling real-world actions without writing a single line of glue code in the client.

---

## 🔭 Overview

| Feature | Details |
|---|---|
| **MCP Server Name** | `DevToBlogPublisher` |
| **Transport** | `stdio` (standard input/output) |
| **Exposed Tool** | `publish_blog_to_devto` |
| **Exposed Prompt** | `blog_post_generator_prompt` |
| **External API** | [Dev.to REST API v1](https://developers.forem.com/api) |
| **Python Version** | `3.12+` |

**What happens end-to-end:**

1. You open **Claude Desktop** and ask Claude to "write and publish a blog post about X".
2. Claude invokes the `blog_post_generator_prompt` template from this MCP server to structure a high-quality blog post.
3. Claude generates the blog content and then calls the `publish_blog_to_devto` tool.
4. The MCP server sends a POST request to Dev.to's API with your article content.
5. The published article URL is returned directly in your Claude conversation.

---

## 🛠️ Tools & Technologies

| Technology | Purpose |
|---|---|
| **Python 3.12** | Core runtime language |
| **[MCP SDK](https://github.com/modelcontextprotocol/python-sdk) (`mcp[cli] >= 1.22.0`)** | FastMCP server framework — defines tools & prompts |
| **`requests >= 2.32.5`** | HTTP client to communicate with the Dev.to REST API |
| **`python-dotenv`** | Loads API keys from `.env` file into the environment |
| **[uv](https://github.com/astral-sh/uv)** | Fast Python package & environment manager |
| **Claude Desktop** | MCP client — connects to this server and invokes its tools |
| **Dev.to REST API** | External service where articles are published |

---

## 🏗️ Workflow Architecture

The following diagram illustrates how all components interact:

```
┌──────────────────────────────────────────────────────────────┐
│                        USER                                  │
│   "Write and publish a blog post about Python async/await"   │
└─────────────────────────┬────────────────────────────────────┘
                          │ Conversation
                          ▼
┌──────────────────────────────────────────────────────────────┐
│                   CLAUDE DESKTOP (MCP Client)                │
│                                                              │
│  1. Reads MCP server config from claude_desktop_config.json  │
│  2. Spawns the MCP server as a child process (stdio)         │
│  3. Discovers available tools & prompts                      │
│  4. Calls blog_post_generator_prompt → generates content     │
│  5. Calls publish_blog_to_devto → publishes article          │
└──────────────┬───────────────────────────────┬───────────────┘
               │ stdio (JSON-RPC 2.0)          │
               ▼                               ▼
┌──────────────────────────┐    ┌──────────────────────────────┐
│   MCP SERVER (devto.py   │    │                              │
│   or dev-server.py)      │    │   Returns article URL        │
│                          │    │   back to Claude Desktop     │
│  ┌────────────────────┐  │    │                              │
│  │ FastMCP Engine     │  │    └──────────────────────────────┘
│  ├────────────────────┤  │
│  │ Tool:              │  │
│  │ publish_blog_to_   │  │
│  │ devto()            │  │
│  ├────────────────────┤  │
│  │ Prompt:            │  │
│  │ blog_post_         │  │
│  │ generator_prompt() │  │
│  └────────┬───────────┘  │
│           │              │
│  Reads DEVTO_API_KEY     │
│  from .env               │
└───────────┬──────────────┘
            │ HTTPS POST /api/articles
            ▼
┌──────────────────────────┐
│   DEV.TO REST API        │
│                          │
│  Creates article         │
│  Returns article URL     │
└──────────────────────────┘
```

### Step-by-Step Flow

```
User Prompt
    │
    ├─► Claude detects intent to publish a blog post
    │
    ├─► [MCP] Invokes `blog_post_generator_prompt(topic=...)`
    │       └─► Returns a structured Markdown prompt template
    │
    ├─► Claude generates full blog post content using the prompt
    │
    ├─► [MCP] Invokes `publish_blog_to_devto(title, body_markdown, tags, ...)`
    │       ├─► Loads DEVTO_API_KEY from environment
    │       ├─► Sends POST https://dev.to/api/articles
    │       └─► Returns success message + article URL
    │
    └─► Claude shows article URL to user in the conversation
```

---

## 🔌 Connecting with Claude Desktop

### Step 1 — Install Claude Desktop

Download and install **Claude Desktop** from [claude.ai/download](https://claude.ai/download).

### Step 2 — Locate the Config File

Open the Claude Desktop configuration file. Its location depends on your OS:

| OS | Path |
|---|---|
| **Windows** | `%APPDATA%\Claude\claude_desktop_config.json` |
| **macOS** | `~/Library/Application Support/Claude/claude_desktop_config.json` |

### Step 3 — Register the MCP Server

Add the following entry inside `claude_desktop_config.json`:

```json
{
  "mcpServers": {
    "DevToBlogPublisher": {
      "command": "uv",
      "args": [
        "run",
        "--directory",
        "C:\\Users\\Admin\\OneDrive\\Documents\\Project-MCP\\MCP-server-Demo",
        "python",
        "devto.py"
      ]
    }
  }
}
```

> **Note:** Replace the directory path with the absolute path to your project folder if it differs.

### Step 4 — Restart Claude Desktop

Fully quit and relaunch Claude Desktop. You should now see the **DevToBlogPublisher** tools available in the Tools panel (hammer icon 🔨).

### Step 5 — Test the Integration

Try asking Claude:

```
Write a blog post about "Getting Started with the Model Context Protocol" 
and publish it as a draft to Dev.to.
```

Claude will automatically invoke the MCP tools and return the draft URL.

---

## ▶️ How to Run the Project

### Prerequisites

- **Python 3.12+** installed
- **uv** package manager installed → [install guide](https://docs.astral.sh/uv/getting-started/installation/)
- A **Dev.to account** and a valid **API key** (see [Dev.to Settings → Account](https://dev.to/settings/account))

---

### 1. Clone the Repository

```bash
git clone <your-repo-url>
cd MCP-server-Demo
```

### 2. Set Up the Virtual Environment

```bash
uv sync
```

This will create a `.venv` directory and install all dependencies defined in `pyproject.toml`.

### 3. Configure Environment Variables

Create a `.env` file in the project root (or edit the existing one):

```bash
cp .env.example .env   # if an example exists, otherwise create manually
```

Then add your Dev.to API key:

```env
DEVTO_API_KEY=your_devto_api_key_here
```

> ⚠️ **Never commit your `.env` file** — it is already in `.gitignore`.

### 4. Run the MCP Server Directly (for testing)

You can run the server manually to verify it starts without errors:

```bash
uv run python devto.py
```

Or use the alternative entry point:

```bash
uv run python dev-server.py
```

The server will start and listen on `stdio`. You won't see a web interface — it's designed to be consumed by an MCP client like Claude Desktop.

### 5. Run with MCP Inspector (optional debugging)

The MCP SDK ships with an inspector tool for testing your server interactively:

```bash
uv run mcp dev devto.py
```

This opens a browser-based UI where you can manually call tools and inspect responses.

---

## 🔐 Environment Variables

| Variable | Required | Description |
|---|---|---|
| `DEVTO_API_KEY` | ✅ Yes | Your personal Dev.to API key used to authenticate article publish requests. |

Obtain your API key from: **Dev.to → Settings → Account → DEV Community API Keys**

---

## 📁 Project Structure

```
MCP-server-Demo/
├── devto.py               # Main MCP server: defines tool & prompt (with dotenv)
├── dev-server.py          # Alternative MCP server entry point
├── pyproject.toml         # Project metadata and dependencies (uv)
├── uv.lock                # Locked dependency versions
├── .env                   # Local environment variables (NOT committed to git)
├── .gitignore             # Git ignore rules
├── .python-version        # Python version pin (3.12)
└── README.md              # This file
```

### Key Files Explained

| File | Description |
|---|---|
| `devto.py` | Full MCP server with both the `publish_blog_to_devto` **tool** and the `blog_post_generator_prompt` **prompt template**. This is the primary server file. |
| `dev-server.py` | Streamlined version of the server that loads `.env` automatically via `python-dotenv`. Useful as an alternative entry point. |
| `pyproject.toml` | Declares the project name (`mcp-server-demo`), Python version constraint, and all runtime dependencies for `uv`. |

---

## 📄 License

This project is open-source and available under the [MIT License](LICENSE).

---

<div align="center">

Built with ❤️ using the [Model Context Protocol](https://modelcontextprotocol.io) · Python 3.12 · FastMCP

</div>
