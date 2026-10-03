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

All requests must be sent as `POST /mcp` with `Content-Type: application/json`.

### Request Format

```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "method": "<method>",
  "params": { ... }
}
```

### Response Format

```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "result": { ... }
}
```

---

## Handshake Methods

### `initialize`

Performs the MCP protocol handshake. Call this first before invoking any tools.

**Request:**
```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "method": "initialize",
  "params": {}
}
```

---

### `tools/list`

Returns the list of all available tools exposed by this MCP server.

**Request:**
```json
{
  "jsonrpc": "2.0",
  "id": 2,
  "method": "tools/list",
  "params": {}
}
```

---

## Tools

### `tools/call`

Invokes a specific tool by name.

**Request:**
```json
{
  "jsonrpc": "2.0",
  "id": 3,
  "method": "tools/call",
  "params": {
    "name": "<tool_name>",
    "arguments": { ... }
  }
}
```

---

## Available Tools

### 1. `full_emi_schedule`

Generates a complete EMI (Equated Monthly Instalment) repayment schedule for every month of the loan tenure.

**Input Arguments:**

| Field          | Type   | Required | Description                                          |
|----------------|--------|----------|------------------------------------------------------|
| `loanAmount`   | string | Yes      | Loan principal amount (e.g. `"500000"`).             |
| `interestRate` | string | Yes      | Annual interest rate as a percentage (e.g. `"8.5"`). |
| `tenureMonths` | string | Yes      | Loan tenure in months (e.g. `"60"`).                 |
| `startDate`    | string | Yes      | Loan start date in `YYYY-MM-DD` format.              |

**Example Request:**
```json
{
  "jsonrpc": "2.0",
  "id": 3,
  "method": "tools/call",
  "params": {
    "name": "full_emi_schedule",
    "arguments": {
      "loanAmount": "500000",
      "interestRate": "8.5",
      "tenureMonths": "60",
      "startDate": "2025-01-01"
    }
  }
}
```

**Example Response:**
```json
{
  "jsonrpc": "2.0",
  "id": 3,
  "result": {
    "GetMonthlyResponse": {
      "Installment_count": 60,
      "Emi": 10224.53,
      "Total_paid": 613471.80,
      "Total_interest": 113471.80,
      "Interest_to_principal_ratio": 0.2934,
      "Schedule": [
        {
          "Month": "2025-01-01",
          "Emi": 10224.53,
          "Interest": 3541.67,
          "Principal": 6682.86,
          "Balance": 493317.14
        }
      ]
    }
  }
}
```

---

### 2. `validate_missing_fields`

Analyses an agent response (containing schema and data) and identifies fields whose value is empty.

**Input Arguments:**

| Field       | Type   | Required | Description                                                              |
|-------------|--------|----------|--------------------------------------------------------------------------|
| `Questions` | string | Yes      | JSON list of question items: `[{"Question":"..."}]`                     |
| `Request`   | string | Yes      | JSON agent response with `DBSchema` and `Data` arrays (see format below).|

**`Request` JSON Format:**
```json
{
  "DBSchema": [
    {
      "Tablename": "CUSTOMER_PERSONAL",
      "AttributeName": "Religion",
      "Type": "string"
    }
  ],
  "Data": [
    {
      "Tablename": "CUSTOMER_PERSONAL",
      "AttributeName": "Religion",
      "Value": "",
      "Question": "What is the religion of the customer?"
    }
  ]
}
```

**Example Request:**
```json
{
  "jsonrpc": "2.0",
  "id": 4,
  "method": "tools/call",
  "params": {
    "name": "validate_missing_fields",
    "arguments": {
      "Questions": "[{\"Question\":\"What is the religion of the customer?\"}]",
      "Request": "{\"DBSchema\":[{\"Tablename\":\"CUSTOMER_PERSONAL\",\"AttributeName\":\"Religion\",\"Type\":\"string\"}],\"Data\":[{\"Tablename\":\"CUSTOMER_PERSONAL\",\"AttributeName\":\"Religion\",\"Value\":\"\",\"Question\":\"What is the religion of the customer?\"}]}"
    }
  }
}
```

## Sample Loan Application Data

Below is a sample customer personal record that can be used as input data for the `validate_missing_fields` tool.
Fields with empty string values are candidates for missing field detection.

```json
{
  "CUSTOMER_PERSONAL": {
    "Id": 1,
    "CustomerId": 1,
    "Salutation": 33,
    "FirstName": "John",
    "MiddleName": "A",
    "LastName": "Smith",
    "FullName": "John A Smith",
    "Gender": "Male",
    "DateOfBirth": "1998-10-24T00:00:00Z",
    "Age": "27",
    "IsMarried": true,
    "Nationality": "",
    "FatherName": "Robert Smith",
    "Religion": "",
    "AdultDependents": 1
  }
}
```

## Author

Eone Development Team.