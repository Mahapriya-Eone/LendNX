# LendNX
LendNX MCP Server helps loan-processing agents reliably identify incomplete loan information and generate accurate repayment schedules, enabling smoother automation of lending workflows.

# LendNX MCP Server

A Model Context Protocol (MCP) server that provides loan processing tools with intelligent missing field detection and EMI schedule generation.

---

## Base URL

```
https://eone-dev.outsystems.app/MCPServer_CustomerApp/rest/MCP/mcp
```

---

## Endpoint

| Method | Path   | Description                                 |
|--------|--------|---------------------------------------------|
| POST   | `/mcp` | Main MCP JSON-RPC 2.0 entry point           |
| GET    | `/mcp` | Returns 405 — use POST for all MCP requests |

---

## Protocol

This server implements the **MCP Streamable HTTP** specification using **JSON-RPC 2.0**.
