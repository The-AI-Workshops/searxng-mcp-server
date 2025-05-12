# SearXNG MCP Server
[![smithery badge](https://smithery.ai/badge/@The-AI-Workshops/searxng-mcp-server)](https://smithery.ai/server/@The-AI-Workshops/searxng-mcp-server)

An MCP sse implementation of the Model Context Protocol (MCP) server integrated with [SearXNG](https://github.com/searxng/searxng) for providing AI agents with powerful, privacy-respecting search capabilities.

---

## Overview

This project demonstrates how to build an MCP server that enables AI agents to perform web searches using a SearXNG instance. It serves as a practical template for creating your own MCP servers, using SearXNG as a backend.

The implementation follows the best practices laid out by Anthropic for building MCP servers, allowing seamless integration with any MCP-compatible client.

---

## Prerequisites

- Python 3.9+
- Access to a running SearXNG instance (local or remote)
- Docker (optional, for containerized deployment)
- [uv](https://github.com/astral-sh/uv) (optional, for fast Python dependency management)
- [Smithery](https://github.com/The-AI-Workshops/smithery) (optional, for MCP server management)

### SearXNG Server (Required)

You must have a SearXNG server running and accessible. The recommended way is via Docker:

```bash
docker run -d --name=searxng -p 32768:8080 -v "/root/searxng:/etc/searxng" \
  -e "BASE_URL=http://0.0.0.0:32768/" \
  -e "INSTANCE_NAME=home" \
  --restart always searxng/searxng
```

- This will run SearXNG on port 32768 and persist configuration in `/root/searxng`.
- The MCP server expects SearXNG to be available at `http://172.17.0.1:32768` by default (see `.env`).

---

## Installation

### Installing via Smithery

To install SearXNG Search Integration Server for Claude Desktop automatically via [Smithery](https://smithery.ai/server/@The-AI-Workshops/searxng-mcp-server):

```bash
npx -y @smithery/cli install @The-AI-Workshops/searxng-mcp-server --client claude
```

### Using uv

Install uv if you don't have it:

```bash
pip install uv
```

Clone this repository:

```bash
git clone https://github.com/The-AI-Workshops/searxng-mcp-server.git
cd searxng-mcp-server/dev/searXNG-mcp
```

Install dependencies:

```bash
uv pip install -r requirements.txt
```

Create a `.env` file based on the provided example:

```bash
nano .env
# Edit .env as needed
```

Configure your environment variables in the `.env` file (see Configuration section).

---

### Using Docker (Recommended)

Build the Docker image:

```bash
docker build -t mcp/searxng-mcp .
```

Create a `.env` file and configure your environment variables.

---
Run the Docker image:

```bash
docker run -d --env-file ./.env -p 32769:32769 mcp/searxng-mcp
```

---

### Using Smithery

[Smithery](https://github.com/The-AI-Workshops/smithery) is a command-line tool for managing AI agent tools and MCP servers.

Install Smithery if you don't have it (see Smithery documentation for various installation methods, e.g., using pipx):
```bash
pipx install smithery
```

Install the SearXNG MCP server using Smithery:
```bash
smithery install @The-AI-Workshops/searxng-mcp-server
```
This will install the server and its dependencies into a dedicated environment managed by Smithery.

After installation, Smithery will provide you with the path to the installed server. You will need to navigate to this directory to configure it. For example, if Smithery installs tools into `~/.smithery/tools/`, the path might be `~/.smithery/tools/The-AI-Workshops/searxng-mcp-server`.

Create a `.env` file in the server's directory by copying the example:
```bash
# Example:
# cd ~/.smithery/tools/The-AI-Workshops/searxng-mcp-server
cp .env.example .env
nano .env
# Edit .env as needed
```
Configure your environment variables in the `.env` file (see Configuration section).

---

## Configuration

The following environment variables can be configured in your `.env` file:

| Variable           | Description                                 | Example                                 |
|--------------------|---------------------------------------------|-----------------------------------------|
| SEARXNG_BASE_URL   | Base URL of your SearXNG instance           | http://172.17.0.1:32768                 |
| HOST               | Host to bind to when using SSE transport    | 0.0.0.0                                 |
| PORT               | Port to listen on when using SSE transport  | 32769                                   |
| TRANSPORT          | Transport protocol (sse or stdio)           | sse                                     |

---

## Running the Server

### Using uv

**SSE Transport**

Set `TRANSPORT=sse` in `.env` then:

```bash
uv run dev/searXNG-mcp/server.py
```

**Stdio Transport**

With stdio, the MCP client itself can spin up the MCP server, so nothing to run at this point.

---

### Using Docker

**SSE Transport**

```bash
docker build -t mcp/searxng-mcp .
docker run --rm -it -p 32769:32769 --env-file dev/searXNG-mcp/.env -v $(pwd)/dev/searXNG-mcp:/app mcp/searxng-mcp
```

- The `-v $(pwd)/dev/searXNG-mcp:/app` mount allows you to live-edit the code and .env file on your host and have changes reflected in the running container.
- The server will be available at `http://localhost:32769/sse`.

**Stdio Transport**

With stdio, the MCP client itself can spin up the MCP server container, so nothing to run at this point.

---

### Running with Smithery

**SSE Transport**

Set `TRANSPORT=sse` in `.env` in the Smithery-installed server directory.
Then, you can typically run the server using the Python interpreter from the virtual environment Smithery created for the tool:
```bash
# Navigate to the server directory, e.g.,
# cd ~/.smithery/tools/The-AI-Workshops/searxng-mcp-server
~/.smithery/venvs/The-AI-Workshops_searxng-mcp-server/bin/python server.py
```
Alternatively, if Smithery provides a direct run command for installed tools (check Smithery documentation):
```bash
smithery run @The-AI-Workshops/searxng-mcp-server
```
The server will be available based on your HOST and PORT settings in `.env` (e.g., `http://localhost:32769/sse`).

**Stdio Transport**

With stdio, the MCP client itself will spin up the server. The client configuration will need to point to the `server.py` script within the Smithery-managed directory, potentially using `smithery exec` or the direct path to the Python interpreter in the tool's virtual environment. See the "Integration with MCP Clients" section for examples.

---

## Integration with MCP Clients

### SSE Configuration

Once you have the server running with SSE transport, you can connect to it using this configuration:

```json
{
  "mcpServers": {
    "searxng": {
      "transport": "sse",
      "url": "http://localhost:32769/sse"
    }
  }
}
```

**Note for Windsurf users:** Use `serverUrl` instead of `url` in your configuration:

```json
{
  "mcpServers": {
    "searxng": {
      "transport": "sse",
      "serverUrl": "http://localhost:32769/sse"
    }
  }
}
```

**Note for n8n users:** Use `host.docker.internal` instead of `localhost` since n8n has to reach outside of its own container to the host machine:

So the full URL in the MCP node would be: `http://host.docker.internal:32769/sse`

Make sure to update the port if you are using a value other than the default 32769.

---

### Python with Stdio Configuration

Add this server to your MCP configuration for Claude Desktop, Windsurf, or any other MCP client:

```json
{
  "mcpServers": {
    "searxng": {
      "command": "python",
      "args": ["dev/searXNG-mcp/server.py"],
      "env": {
        "TRANSPORT": "stdio",
        "SEARXNG_BASE_URL": "http://localhost:32768",
        "HOST": "0.0.0.0",
        "PORT": "32769"
      }
    }
  }
}
```

---

### Docker with Stdio Configuration

```json
{
  "mcpServers": {
    "searxng": {
      "command": "docker",
      "args": ["run", "--rm", "-i",
               "-e", "TRANSPORT",
               "-e", "SEARXNG_BASE_URL",
               "-e", "HOST",
               "-e", "PORT",
               "mcp/searxng-mcp"],
      "env": {
        "TRANSPORT": "stdio",
        "SEARXNG_BASE_URL": "http://localhost:32768",
        "HOST": "0.0.0.0",
        "PORT": "32769"
      }
    }
  }
}
```

---

### Smithery with Stdio Configuration

If you installed the server using Smithery, you can configure your MCP client to run it via stdio. Smithery provides an `exec` command to run executables from within the tool's environment.

```json
{
  "mcpServers": {
    "searxng": {
      "command": "smithery",
      "args": ["exec", "@The-AI-Workshops/searxng-mcp-server", "--", "python", "server.py"],
      // "cwd" (current working directory) might be automatically handled by Smithery.
      // If server.py is in a subdirectory, adjust the python script path e.g., "python", "path/to/server.py"
      "env": {
        "TRANSPORT": "stdio",
        "SEARXNG_BASE_URL": "http://localhost:32768", // Adjust as needed
        "HOST": "0.0.0.0", // Typically not used by stdio server itself but good to set
        "PORT": "32769"  // Typically not used by stdio server itself
      }
    }
  }
}
```
Alternatively, you can find the path to the Python interpreter in the virtual environment created by Smithery (e.g., `~/.smithery/venvs/The-AI-Workshops_searxng-mcp-server/bin/python`) and the path to `server.py` (e.g., `~/.smithery/tools/The-AI-Workshops/searxng-mcp-server/server.py`) and use those directly:
```json
{
  "mcpServers": {
    "searxng": {
      "command": "~/.smithery/venvs/The-AI-Workshops_searxng-mcp-server/bin/python",
      "args": ["~/.smithery/tools/The-AI-Workshops/searxng-mcp-server/server.py"],
      // "cwd" should be the directory containing server.py if not using absolute paths for args,
      // or if server.py relies on relative paths for other files (like .env).
      // Example: "cwd": "~/.smithery/tools/The-AI-Workshops/searxng-mcp-server",
      "env": {
        "TRANSPORT": "stdio",
        "SEARXNG_BASE_URL": "http://localhost:32768"
        // Other necessary env vars from .env can be duplicated here
      }
    }
  }
}
```
Ensure the paths are correct for your Smithery installation and that the `.env` file is discoverable by `server.py` (usually by setting `cwd` to the server's root directory or ensuring `server.py` loads it from an absolute path if Smithery sets one).

---

## Building Your Own Server

This template provides a foundation for building more complex MCP servers. To build your own:

- Add your own tools by creating methods with the `@mcp.tool()` decorator
- Create your own lifespan function to add your own dependencies (clients, database connections, etc.)
- Add prompts and resources as well with `@mcp.resource()` and `@mcp.prompt()`

---

## SearXNG Search Tool Parameters

The `search` tool supports the following parameters (all optional except `q`):

- `q` (required): The search query string.
- `categories`: Comma-separated list of active search categories.
- `engines`: Comma-separated list of active search engines.
- `language`: Code of the language.
- `page`: Search page number (default: 1).
- `time_range`: [day, month, year]
- `format`: [json, csv, rss] (default: json)
- `results_on_new_tab`: [0, 1]
- `image_proxy`: [true, false]
- `autocomplete`: [google, dbpedia, duckduckgo, mwmbl, startpage, wikipedia, stract, swisscows, qwant]
- `safesearch`: [0, 1, 2]
- `theme`: [simple]
- `enabled_plugins`: List of enabled plugins.
- `disabled_plugins`: List of disabled plugins.
- `enabled_engines`: List of enabled engines.
- `disabled_engines`: List of disabled engines.

See the [SearXNG documentation](https://docs.searxng.org/) for more details.

---

## License

MIT License
