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
