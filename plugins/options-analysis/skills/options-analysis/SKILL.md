---
name: options-analysis
description: Use when making structured decisions between multiple options - creates weighted scoring spreadsheet with systematic evaluation across considerations grouped by area
---

# Options Analysis

## Overview

Guide users through creating structured multi-criteria decision analysis (MCDA) spreadsheets using Google Sheets.

**Core principle:** Systematic evaluation with weighted scoring, resumable workflow, and state-driven navigation.

**Announce at start:** "I'm using the options-analysis skill to help you systematically evaluate your options."

**When to use:**
- Comparing multiple options (cloud providers, vendors, architectural approaches)
- Need structured, auditable decision framework
- Want weighted scoring across multiple criteria
- Require justifications for each score

**Output:** Google Spreadsheet with:
- Analysis sheet: Options × Considerations with scores (0-5) and justifications
- Summary sheet: Weighted scores and ranking
- Metadata sheet: Area weights and workflow state

## Prerequisites

This skill requires a Google Sheets MCP server. If not already configured, see Setup Guide below.

## Table of Contents

1. [Setup Guide](#setup-guide) - Install Google Sheets MCP server
2. [Quick Reference](#quick-reference) - State detection and phase mapping
3. [The Workflow](#the-workflow) - Main checklist and phases
4. [Phase Details](#phase-details) - Detailed instructions per phase
5. [State Detection Logic](#state-detection-logic) - How to infer current state
6. [Google Sheets Operations](#google-sheets-operations) - MCP patterns and formulas
7. [Common Mistakes](#common-mistakes) - Anti-patterns and troubleshooting

---

## Setup Guide

### Installing Google Sheets MCP Server

**Check if already configured:**

```bash
# Check for google-sheets MCP server in available tools
# (MCP servers show up as mcp__google-sheets__* tools)
```

If you see `mcp__google-sheets__*` tools available, skip to [The Workflow](#the-workflow).

**Install Google Sheets MCP Server:**

Option 1: Using npx (Node.js required):
```bash
npx -y google-sheets-mcp
```

Option 2: Using uvx (Python, recommended):
```bash
uvx mcp-google-sheets@latest
```

**Configure in Claude Code:**

Add to your Claude Code MCP configuration file:

For npx version:
```json
{
  "mcpServers": {
    "google-sheets": {
      "command": "npx",
      "args": ["-y", "google-sheets-mcp"]
    }
  }
}
```

For uvx version:
```json
{
  "mcpServers": {
    "google-sheets": {
      "command": "uvx",
      "args": ["mcp-google-sheets@latest"]
    }
  }
}
```

### Authentication Setup

**Recommended: Service Account**

1. Go to Google Cloud Console: https://console.cloud.google.com/
2. Create new project or select existing
3. Enable Google Sheets API
4. Create Service Account credentials
5. Download JSON key file
6. Set environment variable:
   ```bash
   export GOOGLE_APPLICATION_CREDENTIALS="/path/to/service-account-key.json"
   ```
7. Share spreadsheets with service account email (found in JSON key)

**Alternative: OAuth 2.0**

Follow MCP server's OAuth setup instructions (more complex, better for personal use).

### Verification

Test the connection:

1. List available MCP tools - should see `mcp__google-sheets__*` functions
2. Try reading a test spreadsheet
3. If errors occur, check:
   - MCP server is running
   - Authentication credentials are valid
   - Service account has access to spreadsheet

---
