# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
# Install / reinstall the package
uv pip install -e .

# Start the MCP server
uv run main.py

# Run all tests
uv run pytest

# Run a single test
uv run pytest tests/test_document.py::TestBinaryDocumentToMarkdown::test_binary_document_to_markdown_with_docx
```

## Architecture

This project is an MCP server built with **FastMCP**. Tools are plain Python functions that live in the `tools/` package and are registered with the server in `main.py`.

**Flow:** `main.py` creates a `FastMCP` instance, imports individual tool functions from `tools/`, and registers each one with `mcp.tool()(fn)`. The server then exposes them over the MCP protocol when run.

**`tools/document.py`** — helper utilities (not yet registered as MCP tools). Currently provides `binary_document_to_markdown`, which wraps the `markitdown` library to convert raw binary data from DOCX or PDF files into markdown text.

**`tools/math.py`** — example MCP tool (`add`). Use this as a reference for the tool pattern.

## Defining MCP Tools

Tools are regular Python functions. The docstring and `Field` descriptions are surfaced directly to the AI assistant, so they must be thorough.

**Registration** — import the function in `main.py` and register it:

```python
from tools.my_module import my_function
mcp.tool()(my_function)
```

**Parameter descriptions** — use `pydantic.Field` as the default value for each parameter:

```python
from pydantic import Field

def my_tool(
    param1: str = Field(description="What this parameter represents"),
    param2: int = Field(description="What this parameter controls"),
) -> str:
    ...
```

**Docstring structure** — follow this four-part format (see `tools/math.py` for a complete example):

1. One-line summary
2. Prose explanation of what the tool does
3. `When to use:` / `When not to use:` bullet list
4. `Examples:` block with `>>> input` / `output` pairs

**Return types** — always annotate the return type; FastMCP uses it to validate and describe the tool output.

**Binary document handling** — when a tool needs to process a document, accept `binary_data: bytes` and `file_type: str`, then delegate to `binary_document_to_markdown` from `tools/document.py`.

## Tests

Test fixtures (`.docx`, `.pdf`) live in `tests/fixtures/`. Tests read fixtures as raw bytes and pass them into the tool functions directly — no MCP layer involved in unit tests.
