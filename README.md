📁 Technical Guide: Building a Persistent CCSP AI Tutor
=======================================================

**Project Name:** ccsp-mcp-server

**Author:** Mbong Hudson Ekwoge

**Stack:** Python 3.12+, FastMCP SDK, uv Package Manager

1\. Overview
------------

This guide provides the blueprints for connecting **Claude Desktop** to a local knowledge base (Markdown) using the **Model Context Protocol (MCP)**. This architecture allows the AI to surgically retrieve exam content without exhausting your token window or requiring manual file uploads.

2\. Prerequisites
-----------------

Before starting, ensure you have the following installed on your Mac:

*   **Claude Desktop App**
    
*   **uv:** The high-speed Python package manager.

```
curl -LsSf https://astral.sh/uv/install.sh | sh
```
3\. Environment Setup
---------------------

Create a dedicated directory for your server. Using a clean environment prevents dependency conflicts.
```
# Create project folder
mkdir ~/ccsp-mcp-server && cd ~/ccsp-mcp-server

# Initialize project and add MCP SDK
uv init
uv add "mcp[cli]"
```
4\. The Server Logic (`server.py`)
--------------------------------

Create a file named `server.py` in your project folder. This script defines how the AI "sees" and "searches" your notes.

> **Note:** Replace YOUR\_USER\_NAME with your actual macOS username.
```
from mcp.server.fastmcp import FastMCP
import os

# Initialize the server
mcp = FastMCP("CCSP-Knowledge-Base")

# Absolute path to your notes file
NOTES_PATH = "/Users/hudson/Documents/ccsp_notes.md"

@mcp.resource("note://ccsp/full")
def get_ccsp_notes() -> str:
    """Reads the entire CCSP notes file into context."""
    if not os.path.exists(NOTES_PATH):
        return f"Error: CCSP notes file not found at {NOTES_PATH}"
    with open(NOTES_PATH, "r", encoding="utf-8") as f:
        return f.read()

@mcp.tool()
def search_ccsp_notes(keyword: str) -> str:
    """
    Search your CCSP notes for a specific keyword or domain.
    Returns the lines containing the keyword.
    """
    if not os.path.exists(NOTES_PATH):
        return f"Error: CCSP notes file not found at {NOTES_PATH}"
    
    results = []
    with open(NOTES_PATH, "r", encoding="utf-8") as f:
        for i, line in enumerate(f, 1):
            if keyword.lower() in line.lower():
                results.append(f"Line {i}: {line.strip()}")
    
    if not results:
        return f"No mentions of '{keyword}' found in notes."
    
    return "\n".join(results)

if __name__ == "__main__":
    mcp.run(transport="stdio")
```
5\. Claude Desktop Configuration
--------------------------------

To "plug in" the server, you must edit your global Claude configuration file.

**File Location:** `~/Library/Application Support/Claude/claude\_desktop\_config.json`

### The Configuration Payload:

Run `which uv` in your terminal to get your absolute path (usually `/Users/<YOUR_USER_NAME>/.local/bin/uv`).
```
{
  "mcpServers": {
    "ccsp-tutor": {
      "command": "/Users/<YOUR_USER_NAME>/.local/bin/uv",
      "args": [
        "run",
        "--directory",
        "/Users/<YOUR_USER_NAME>/Downloads/ccsp-mcp-server",
        "server.py"
      ]
    }
  },
  "preferences": {
    "coworkScheduledTasksEnabled": false,
    "ccdScheduledTasksEnabled": false,
    "coworkWebSearchEnabled": true,
    "sidebarMode": "chat"
  }
}
```
7\. Verification Prompts
------------------------

Once the green plug icon appears in Claude, test your server with:

1.  _"What tools do you have available via the ccsp-tutor?"_
    
2.  _"Search my CCSP notes for 'Data Custodian' and explain the cloud-specific role."_
    

### Pro-Tip for Exam Prep

The **Search Tool** is significantly more cost-efficient than the **Resource** (Full Read). Only ask the AI to "read the whole file" when you need a broad summary. For specific definitions, use the search\_ccsp\_notes tool to save on token costs and increase response speed.
