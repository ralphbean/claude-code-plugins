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

## Quick Reference

| Current State | Detected By | Next Phase |
|---------------|-------------|------------|
| No spreadsheet | User hasn't provided URL | Create new spreadsheet |
| Empty spreadsheet | No sheets named "Analysis", "Summary", "Metadata" | Define options |
| Has Metadata sheet | Read workflow checklist from Metadata | Jump to incomplete phase |
| `options_defined: false` | Metadata.B8 = FALSE | Define Options (Phase 2) |
| `areas_defined: false` | Metadata.B9 = FALSE | Define Areas & Weights (Phase 3) |
| `considerations_defined: false` | Metadata.B10 = FALSE | Define Considerations (Phase 4) |
| `scoring_complete: false` | Metadata.B11 = FALSE | Scoring (Phase 5) |
| `summary_created: false` | Metadata.B12 = FALSE | Summary Generation (Phase 6) |
| All checklist items TRUE | All Metadata workflow flags = TRUE | Maintenance Mode (Phase 7) |

---

## State Detection Logic

When the skill is invoked, follow this sequence:

### Step 1: Get Spreadsheet Reference

Ask user: "Do you have an existing options analysis spreadsheet, or should I create a new one?"

**If new:**
- Prompt for analysis name/title
- Create new spreadsheet with that title
- Proceed to Phase 2 (Define Options)

**If existing:**
- Prompt for spreadsheet URL or ID
- Extract spreadsheet ID from URL (format: `https://docs.google.com/spreadsheets/d/{SPREADSHEET_ID}/edit`)
- Proceed to Step 2

### Step 2: Read Spreadsheet Structure

Use MCP tool to list sheets in spreadsheet.

**Check for key sheets:**
- "Analysis" sheet exists?
- "Summary" sheet exists?
- "Metadata" sheet exists?

**If no sheets exist:** Fresh spreadsheet, go to Phase 2 (Define Options)

**If Metadata sheet exists:** Read workflow state (Step 3)

**If only Analysis exists, no Metadata:** Corrupted state - offer to create Metadata sheet and infer state from Analysis structure

### Step 3: Read Metadata Sheet Workflow State

Metadata sheet structure:
```
     A                     B
7  Workflow State
8  options_defined       TRUE/FALSE
9  areas_defined         TRUE/FALSE
10 considerations_defined TRUE/FALSE
11 scoring_complete      TRUE/FALSE
12 summary_created       TRUE/FALSE
```

Read cells B8:B12 to determine which phases are complete.

### Step 4: Display Current State and Menu

Example output:
```
Options Analysis Status:
✓ Options defined (3 options: AWS, Azure, GCP)
✓ Areas defined (4 areas with weights)
✓ Considerations defined (18 considerations across areas)
○ Scoring in progress (42 of 54 cells scored - 78%)
○ Summary not yet created

What would you like to do?
1. Continue scoring (12 cells remaining)
2. Add new option
3. Add new consideration or area
4. Modify area weights
5. Review current scores
6. Generate summary (requires all scoring complete)
7. Start fresh analysis (will archive current work)
```

Use AskUserQuestion tool to present menu with options.

### Step 5: Route to Appropriate Phase

Based on user selection and current state, jump to the relevant phase section below.

---
