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

## The Workflow

When starting this skill, create a TodoWrite checklist to track progress:

```markdown
Options Analysis Workflow:
- [ ] Phase 1: State Detection & Orientation
- [ ] Phase 2: Define Options
- [ ] Phase 3: Define Areas & Weights
- [ ] Phase 4: Define Considerations
- [ ] Phase 5: Scoring
- [ ] Phase 6: Summary Generation
- [ ] Phase 7: Maintenance (ongoing)
```

Copy this checklist to TodoWrite at the start of each session. Update task status as you progress through phases.

### Phase Overview

| Phase | Goal | Output |
|-------|------|--------|
| 1. State Detection | Determine current analysis state | Navigation menu |
| 2. Define Options | Set up options being evaluated | Analysis sheet with option columns |
| 3. Define Areas & Weights | Create decision areas with importance weights | Metadata sheet with normalized weights |
| 4. Define Considerations | Add specific criteria under each area | Analysis sheet with colored, grouped rows |
| 5. Scoring | Collect scores and justifications for each cell | Populated Analysis sheet |
| 6. Summary Generation | Calculate weighted scores and ranking | Summary sheet with formulas |
| 7. Maintenance | Add options/considerations, modify weights | Updated analysis |

Phases 2-6 can be partially completed and resumed. Phase 7 is entered after initial analysis is complete.

---
## Phase Details

### Phase 2: Define Options

**Goal:** Establish the options being evaluated (e.g., AWS, Azure, GCP).

**Prerequisites:** Spreadsheet created (empty or has ID)

**Steps:**

#### Step 1: Prompt for options

Ask user: "What options are you evaluating? Please list them separated by commas."

Example user response: "AWS, Azure, Google Cloud Platform"

Parse response into array: `["AWS", "Azure", "Google Cloud Platform"]`

#### Step 2: Create/update Analysis sheet

**If Analysis sheet doesn't exist:**

Use MCP tool to create sheet named "Analysis"

**Set up header row (Row 1):**

| A | B | C | D |
|---|---|---|---|
| Consideration | AWS | Azure | Google Cloud Platform |

Write to cells:
- A1: "Consideration"
- B1: First option name
- C1: Second option name
- ... (one column per option)

**Formatting:**
- Row 1: Bold, freeze row (so it stays visible when scrolling)
- Font size: 11pt
- Background: Light gray (#F3F3F3)

#### Step 3: Create Metadata sheet

Create sheet named "Metadata"

**Write initial structure:**

```
     A                     B        C          D
1  Options Analysis Metadata
2
3  Options:
4  [List options here]
5
6  Areas & Weights:
7  (To be defined in Phase 3)
8
9  Workflow State:
10 options_defined       TRUE
11 areas_defined         FALSE
12 considerations_defined FALSE
13 scoring_complete      FALSE
14 summary_created       FALSE
```

Write cells:
- A1: "Options Analysis Metadata" (Bold, size 14)
- A3: "Options:"
- A4: Join options with ", " (e.g., "AWS, Azure, Google Cloud Platform")
- A6: "Areas & Weights:"
- A9: "Workflow State:"
- A10: "options_defined", B10: "TRUE"
- A11: "areas_defined", B11: "FALSE"
- A12: "considerations_defined", B12: "FALSE"
- A13: "scoring_complete", B13: "FALSE"
- A14: "summary_created", B14: "FALSE"

#### Step 4: Update TodoWrite and report

Mark Phase 2 as complete in TodoWrite.

Report to user:
```
✓ Phase 2 Complete: Options defined

Analysis sheet created with [N] options: [option names]
Metadata sheet initialized with workflow state.

Next: Define decision areas and their weights (Phase 3)
```

Proceed to Phase 3 or return to menu based on user preference.

---
### Phase 3: Define Areas & Weights

**Goal:** Create conceptual decision areas (e.g., Reliability, Cost) and assign importance weights.

**Prerequisites:** Phase 2 complete (options defined)

**Steps:**

#### Step 1: Prompt for areas

Ask user: "What are the key decision areas you want to evaluate? These group related considerations together."

Examples to suggest:
- Technical: "Reliability, Performance, Security, Scalability"
- Business: "Cost, Vendor Support, Ecosystem, Compliance"
- User Experience: "Usability, Documentation, Community, Tooling"

Parse response into array of area names.

#### Step 2: Prompt for weights

For each area, ask: "On a scale of relative importance, how would you weight [Area Name]? You can use percentages (e.g., 30%) or relative numbers (e.g., 3). I'll normalize them afterward."

Collect weights for all areas.

**Normalization:**

Example inputs: Reliability: 40, Cost: 30, Security: 20, Usability: 10

Sum: 100 (already normalized)

If sum ≠ 100, normalize:
- Reliability: 40/100 * 100 = 40%
- Cost: 30/100 * 100 = 30%
- Security: 20/100 * 100 = 20%
- Usability: 10/100 * 100 = 10%

#### Step 3: Write areas to Metadata sheet

Update Metadata sheet starting at row 7:

```
     A              B
6  Areas & Weights:
7  Reliability     40
8  Cost            30
9  Security        20
10 Usability       10
```

For each area:
- Column A: Area name
- Column B: Normalized weight (as number, e.g., 40 not 40%)

#### Step 4: Choose colors for areas

Assign distinct, pleasant background colors to each area for visual grouping:

| Area Index | Color Name | Hex Code |
|------------|------------|----------|
| 1 | Light Blue | #CFE2F3 |
| 2 | Light Green | #D9EAD3 |
| 3 | Light Yellow | #FFF2CC |
| 4 | Light Orange | #F4CCCC |
| 5 | Light Purple | #D9D2E9 |
| 6 | Light Teal | #D0E0E3 |

Cycle through colors if more than 6 areas.

Store color mapping in memory for Phase 4.

#### Step 5: Update workflow state

Update Metadata!B11 to "TRUE" (areas_defined)

#### Step 6: Report and proceed

Mark Phase 3 complete in TodoWrite.

Report:
```
✓ Phase 3 Complete: Areas & Weights defined

4 decision areas created:
- Reliability (40%)
- Cost (30%)
- Security (20%)
- Usability (10%)

Weights normalized to sum to 100%.

Next: Define specific considerations within each area (Phase 4)
```

Proceed to Phase 4 or return to menu.

---
### Phase 4: Define Considerations

**Goal:** Add specific evaluation criteria under each decision area, with visual grouping.

**Prerequisites:** Phase 3 complete (areas and weights defined)

**Steps:**

#### Step 1: Read areas from Metadata

Read Metadata sheet rows 7+ to get list of areas and their assigned colors.

#### Step 2: For each area, prompt for considerations

For each area, ask: "What specific considerations fall under [Area Name]? List them separated by commas."

Example for "Reliability" area:
User response: "Multi-region support, Disaster recovery options, SLA guarantees, Monitoring capabilities"

Parse into array: `["Multi-region support", "Disaster recovery options", "SLA guarantees", "Monitoring capabilities"]`

Repeat for all areas.

#### Step 3: Build Analysis sheet structure

Start writing to Analysis sheet at row 3 (row 1 is header, row 2 blank for spacing).

Current row tracker: `current_row = 3`

**For each area:**

1. **Write area header row:**
   - A[current_row]: Area name + weight, e.g., "Reliability (40%)"
   - Format:
     - Bold font
     - Font size: 12pt
     - Background color: Area's assigned color (from Phase 3)
     - Merge cells A[current_row]:[last_option_column][current_row]
   - Increment: `current_row += 1`

2. **Write consideration rows:**
   - For each consideration in this area:
     - A[current_row]: "  ├─ " + consideration name (e.g., "  ├─ Multi-region support")
     - Use "  └─ " for the last consideration in the area
     - Format:
       - Normal font (not bold)
       - Background color: Lighter shade of area color (add 30% white overlay)
     - For score columns (B through last option):
       - Leave cell empty (for scoring in Phase 5)
       - Set data validation: Whole number between 0 and 5
       - Set conditional formatting:
         - 0-1: Red background (#F4CCCC)
         - 2-3: Yellow background (#FFF2CC)
         - 4-5: Green background (#D9EAD3)
     - Increment: `current_row += 1`

3. **Add blank row for spacing:**
   - Increment: `current_row += 1`

#### Step 4: Example structure

After this phase, Analysis sheet should look like:

```
     A                          B      C       D
1  Consideration              AWS   Azure   GCP
2
3  Reliability (40%)          [merged across all columns, light blue bg]
4    ├─ Multi-region support  [empty] [empty] [empty]
5    ├─ Disaster recovery     [empty] [empty] [empty]
6    ├─ SLA guarantees        [empty] [empty] [empty]
7    └─ Monitoring            [empty] [empty] [empty]
8
9  Cost (30%)                 [merged across all columns, light green bg]
10   ├─ Total cost of ownership [empty] [empty] [empty]
11   └─ Licensing model       [empty] [empty] [empty]
12
...
```

All empty cells have data validation (0-5) and conditional formatting.

#### Step 5: Update Metadata workflow state

Update Metadata!B12 to "TRUE" (considerations_defined)

#### Step 6: Calculate total scoring cells

Count total empty score cells: `num_options * num_considerations`

Store this in memory for progress tracking in Phase 5.

#### Step 7: Report and proceed

Mark Phase 4 complete in TodoWrite.

Report:
```
✓ Phase 4 Complete: Considerations defined

18 considerations added across 4 areas:
- Reliability: 4 considerations
- Cost: 2 considerations
- Security: 8 considerations
- Usability: 4 considerations

Total scoring cells to complete: 54 (18 considerations × 3 options)

Data validation and conditional formatting applied to all score cells.

Next: Score each option for each consideration (Phase 5)
```

Proceed to Phase 5 or return to menu.

---
### Phase 5: Scoring

**Goal:** Collect scores (0-5) and justifications for each option-consideration pair.

**Prerequisites:** Phase 4 complete (considerations defined)

**Steps:**

#### Step 1: Identify unscored cells

Scan Analysis sheet for empty cells in score columns (columns B onward, excluding area header rows).

Build list of unscored cells with their coordinates and labels:
- Cell: B4
- Option: "AWS" (from column B header)
- Consideration: "Multi-region support" (from column A, removing tree chars)
- Area: "Reliability" (from nearest area header above)

#### Step 2: Display progress

Calculate and display progress:
```
Scoring Progress: 42 of 54 cells scored (78%)

Remaining cells by area:
- Reliability: 3 cells
- Cost: 5 cells
- Security: 2 cells
- Usability: 2 cells
```

#### Step 3: Prompt for scores

**For each unscored cell:**

Use AskUserQuestion or direct prompt:

"Score [Option] for [Consideration] (under [Area]):

Scale:
0 = Poor/Not supported
1 = Minimal/Barely adequate
2 = Below average
3 = Average/Acceptable
4 = Good/Above average
5 = Excellent/Best in class

Score (0-5): "

Wait for user input (validate it's a number 0-5).

**Then prompt for justification:**

"Provide a brief justification for this score (this will be saved as a cell comment): "

Wait for user input (1-2 sentences recommended).

#### Step 4: Write score and comment

- Write the numeric score to the cell
- Add the justification as a cell comment/note using Google Sheets MCP

Example MCP call (pseudocode):
```
mcp__google-sheets__write_cell(
  spreadsheet_id=current_spreadsheet,
  sheet="Analysis",
  cell="B4",
  value=4
)

mcp__google-sheets__add_comment(
  spreadsheet_id=current_spreadsheet,
  sheet="Analysis",
  cell="B4",
  comment="Strong multi-region support with automated failover. Tested in production."
)
```

#### Step 5: Update progress

After each score, recalculate and display progress:
```
✓ Scored AWS for Multi-region support: 4/5

Progress: 43 of 54 cells scored (80%)
Remaining: 11 cells
```

#### Step 6: Handle interruptions

User can pause anytime. Skill should:
- Save all scores entered so far
- Return to menu
- When resumed, detect which cells are still empty and continue from there

#### Step 7: Completion detection

When all cells are scored (no empty score cells remain):

Update Metadata!B13 to "TRUE" (scoring_complete)

Mark Phase 5 complete in TodoWrite.

Report:
```
✓ Phase 5 Complete: All scoring finished

54 of 54 cells scored (100%)

All options have been evaluated across all considerations.
Justifications saved as cell comments.

Next: Generate summary with weighted scoring (Phase 6)
```

Proceed to Phase 6 or return to menu.

---
### Phase 6: Summary Generation

**Goal:** Create Summary sheet with weighted scoring formulas and ranking.

**Prerequisites:** Phase 5 complete (all cells scored)

**Steps:**

#### Step 1: Create Summary sheet

Use MCP tool to create sheet named "Summary"

#### Step 2: Set up Summary sheet structure

**Row 1 - Headers:**

| A | B | C | D |
|---|---|---|---|
| Option | Raw Score | Weighted Score | Rank |

Format: Bold, background #F3F3F3, freeze row

#### Step 3: Write option names

For each option (from Analysis!B1, C1, D1...):

Write to Summary column A, starting row 2:
- A2: First option name (e.g., "AWS")
- A3: Second option name
- ... etc

#### Step 4: Calculate Raw Score (simple average)

For each option row in Summary:

**Formula for Raw Score (column B):**

Example for AWS (Summary!B2):
```
=AVERAGE(Analysis!B:B)
```

BUT we need to exclude:
- Header row (row 1)
- Area header rows (merged cells)
- Empty cells

**Better formula using AVERAGEIF:**

```
=AVERAGEIF(Analysis!B:B, ">-1", Analysis!B:B)
```

This averages all numeric values in column B of Analysis sheet.

Write this formula to Summary!B2 (for first option).

Adjust column reference for each option (B for first, C for second, etc.).

#### Step 5: Calculate Weighted Score

**This is the critical formula to avoid consideration-count bias.**

Read areas and weights from Metadata sheet (rows 7+).

For each area:
1. Identify which rows in Analysis belong to this area
2. Calculate AVERAGE of scores for this option in those rows
3. Multiply by area weight
4. Sum across all areas

**Formula structure for AWS (Summary!C2):**

```excel
=SUMPRODUCT(
  Metadata!B7:B10,  // Area weights (40, 30, 20, 10)
  {
    AVERAGE(Analysis!B4:B7),   // Reliability scores (rows 4-7)
    AVERAGE(Analysis!B10:B11), // Cost scores (rows 10-11)
    AVERAGE(Analysis!B14:B21), // Security scores (rows 14-21)
    AVERAGE(Analysis!B24:B27)  // Usability scores (rows 24-27)
  }
) / 100
```

**Dynamic formula generation:**

Since row ranges depend on how many considerations per area, we need to:

1. Read Analysis sheet to determine area row ranges
2. Build the array formula dynamically
3. Write formula to cell

**Pseudocode:**
```
areas = read_metadata_areas()  // [(name, weight, color), ...]
area_ranges = []

for area in areas:
  start_row = find_area_header_row(area.name) + 1
  end_row = start_row + count_considerations_in_area(area) - 1
  area_ranges.append(f"AVERAGE(Analysis!B{start_row}:B{end_row})")

formula = f"=SUMPRODUCT(Metadata!B7:B{6+len(areas)}, {{{','.join(area_ranges)}}})/100"

write_cell(Summary!C2, formula)
```

Repeat for each option, adjusting column (B→C→D...).

#### Step 6: Add Rank formula

For each option in Summary:

**Formula for Rank (column D):**

```
=RANK(C2, $C$2:$C$[last_row], 0)
```

This ranks based on Weighted Score (column C), descending order (0 = highest is rank 1).

Example for AWS (Summary!D2):
```
=RANK(C2, $C$2:$C$4, 0)
```

Assuming 3 options (rows 2-4).

#### Step 7: Apply conditional formatting to Rank

Format Rank column with colors:
- Rank 1: Green background (#D9EAD3), bold
- Rank 2: Yellow background (#FFF2CC)
- Rank 3+: Light red background (#F4CCCC)

#### Step 8: Format numbers

- Raw Score: 2 decimal places (e.g., 3.45)
- Weighted Score: 2 decimal places (e.g., 72.30)
- Rank: Whole number (e.g., 1)

#### Step 9: Update Metadata workflow state

Update Metadata!B14 to "TRUE" (summary_created)

All workflow states now TRUE - analysis complete!

#### Step 10: Report results

Mark Phase 6 complete in TodoWrite.

Display summary results:
```
✓ Phase 6 Complete: Summary generated

Final Rankings:
1. AWS - Weighted Score: 78.50 (Raw: 4.20)
2. Azure - Weighted Score: 71.20 (Raw: 3.85)
3. Google Cloud Platform - Weighted Score: 65.80 (Raw: 3.50)

Summary sheet created with:
- Raw scores (simple average across all considerations)
- Weighted scores (area-weighted average)
- Automatic ranking

Formulas will update automatically if scores change.

Analysis complete! You can now:
- Review the Summary sheet for final rankings
- Add more options or considerations (Phase 7 - Maintenance)
- Share the spreadsheet with stakeholders
```

Proceed to Phase 7 (Maintenance Mode) or end session.

---
