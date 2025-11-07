# Options Analysis Skill Design

**Date:** 2025-11-07
**Author:** Ralph Bean
**Status:** Approved for Implementation

## Overview

A Claude Code skill that guides users through creating structured, multi-criteria decision analysis (MCDA) spreadsheets using Google Sheets. The skill enables systematic evaluation of multiple options across weighted considerations grouped into conceptual areas.

## Use Case

Decision-makers need to evaluate options (e.g., cloud providers, vendors, architectural approaches) across multiple criteria in a structured, auditable way. The skill produces a Google Spreadsheet with:

- Systematic scoring framework (0-5 scale)
- Justifications captured as cell comments
- Area-based grouping of considerations
- Weighted scoring that avoids bias from consideration count
- Clear visual organization with color coding
- Resumable workflow that maintains state

## Architecture: Monolithic State Machine Skill

**Approach:** Single comprehensive SKILL.md file containing complete workflow from MCP setup through spreadsheet creation, scoring, and summary generation.

**Rationale:**
- Self-contained, easy to install (single skill)
- State detection inherently centralized
- Matches pattern of existing `searching-slack-history` skill
- Simpler user experience than multi-skill coordination
- Easier to test with subagents (per `writing-skills` guidance)

**Trade-off:** Larger file (~500 lines) offset by clear section organization and table of contents.

## Key Design Decisions

### 1. State Management
- **Metadata Sheet:** Third sheet stores workflow state, area weights, and checklist status
- **State Detection:** Reads spreadsheet structure and Metadata to infer current phase
- **Resumability:** Users can return to partially-complete analyses anytime
- **Flexible Navigation:** Checklist-based rather than strictly linear workflow

### 2. Google Sheets MCP Integration
- **Setup Included:** Skill guides through MCP server installation/configuration
- **Server Options:** Supports npx google-sheets-mcp or other compatible servers
- **Authentication:** Guides through Service Account or OAuth setup
- **Operations Used:** Create sheets, read/write cells, format cells, add comments, set data validation

### 3. Spreadsheet Structure

#### Analysis Sheet
```
     A                  B      C       D
1  [Consideration]    [AWS]  [Azure] [GCP]
2
3  Reliability (40%)
4  ├─ Multi-region     4      3       5
5  ├─ Monitoring       5      4       4
6
7  Cost (30%)
8  ├─ TCO              3      4       3
9  └─ Licensing        4      3       5
```

**Columns:**
- Column A: Area names with weights (bold, colored, merged) and considerations (indented with tree characters)
- Columns B+: One column per option containing numeric scores (0-5)
- Justifications: Stored as Google Sheets cell comments on score cells

**Formatting:**
- Area rows: Bold, colored background (unique color per area), merged across all columns
- Consideration rows: Normal text, lighter shade of parent area's color
- Score cells: Center-aligned, data validation (0-5 integers), conditional formatting (red→yellow→green)

#### Summary Sheet
```
      A           B          C          D
1  [Option]  [Raw Score] [Weighted] [Rank]
2   AWS         4.2         78.5       1
3   Azure       3.8         71.2       2
4   GCP         3.5         65.8       3
```

**Formulas:**
- Raw Score: Simple average across all considerations for that option
- Weighted Score: Area-weighted average using SUMPRODUCT
- Rank: Automatic ranking based on weighted score

#### Metadata Sheet
```
     A                B
1  Areas & Weights
2  Reliability      40
3  Cost             30
4  Security         20
5  Usability        10
6
7  Workflow State
8  options_defined  TRUE
9  areas_defined    TRUE
10 scoring_complete FALSE
```

Stores area weights (for formula reference) and workflow checklist state.

### 4. Weighted Scoring Formula

**Critical requirement:** Area weights must not be biased by number of considerations per area.

**Solution:** Calculate average per area first, then apply weights:

```excel
For AWS weighted score (Summary!C2):
= SUMPRODUCT(
    Metadata!B2:B5,  // Area weights (40, 30, 20, 10)
    {
      AVERAGE(Analysis!B4:B5),   // Reliability scores
      AVERAGE(Analysis!B8:B9),   // Cost scores
      AVERAGE(Analysis!B11:B13), // Security scores (3 considerations)
      AVERAGE(Analysis!B15:B15)  // Usability scores (1 consideration)
    }
  ) / 100
```

**Formula generation:** Skill generates this programmatically by reading area ranges from Metadata sheet.

**Validation:** Each area contributes proportionally to its weight regardless of consideration count.

### 5. Extensibility

**Adding Options:**
- Append new column to right of existing options (preserves column positions)
- Update Summary sheet formulas to include new option
- All area-based formulas remain valid

**Adding Considerations:**
- Insert row within appropriate area group
- Apply area's color formatting
- Summary formulas automatically expand if using named ranges (or regenerate formula)

**Modifying Area Weights:**
- Update Metadata sheet weights
- Regenerate Summary sheet formulas
- Option: Support in-place formula updates via cell reference

## Workflow Phases

### Phase 0: MCP Server Setup (Conditional)
1. Detect if Google Sheets MCP server configured
2. If not, provide installation command: `npx -y google-sheets-mcp`
3. Guide through authentication (Service Account recommended)
4. Verify connection by listing test spreadsheet

### Phase 1: State Detection & Orientation
1. Prompt for spreadsheet URL/ID or offer to create new
2. If existing spreadsheet:
   - Read Analysis, Summary, Metadata sheets
   - Parse Metadata to determine checklist state
   - Display progress: "Options defined ✓, Areas defined ✓, 45 of 69 cells scored"
3. Present navigation menu:
   - Continue from current phase
   - Add new option/consideration
   - Review summary
   - Start fresh

### Phase 2: Define Options
1. Prompt: "What options are you evaluating?"
2. Create/update Analysis sheet with option columns
3. Write to Metadata: `options: ["AWS", "Azure", "GCP"]`
4. Update checklist: `options_defined: true`

### Phase 3: Define Areas & Weights
1. Prompt: "What are the key decision areas?"
2. For each area: "Weight for [area]? (percentage or relative number)"
3. Normalize weights to sum to 100%
4. Write to Metadata sheet with normalized values
5. Create area header rows in Analysis sheet (colored, merged)
6. Update checklist: `areas_defined: true`

### Phase 4: Define Considerations
1. For each area: "What considerations fall under [area]?"
2. Add consideration rows under appropriate area header
3. Apply color formatting (lighter shade of area color)
4. Add tree characters (├─ and └─) for visual hierarchy
5. Set data validation on score cells (0-5 whole numbers)
6. Update checklist: `considerations_defined: true`

### Phase 5: Scoring
1. Identify unscored cells (empty cells in option columns)
2. Display progress: "23 of 69 cells scored (33%)"
3. For each unscored cell:
   - Prompt: "Score [Option] for [Consideration]? (0-5)"
   - Prompt: "Justification for this score?"
   - Write score to cell
   - Add justification as cell comment
4. Update checklist: `scoring_complete: true` when all cells scored

### Phase 6: Summary Generation
1. Calculate raw scores (simple average per option)
2. Generate weighted score formula using SUMPRODUCT and area weights
3. Create Summary sheet with Option, Raw Score, Weighted Score, Rank columns
4. Apply conditional formatting (rank-based colors)
5. Add sparklines or charts (optional)
6. Update checklist: `summary_created: true`

### Phase 7: Maintenance Mode
Present menu when analysis complete:
1. Add new option → Insert column, update formulas, mark cells for scoring
2. Add new consideration → Prompt for area, insert row, update validation
3. Modify area weights → Update Metadata, recalculate Summary formulas
4. Re-score cells → Return to scoring mode for specific cells
5. Export/share → Provide shareable link and permissions guidance

## Skill Structure (SKILL.md Organization)

```markdown
---
name: options-analysis
description: Use when making structured decisions...
---

# Options Analysis

## Overview
- Core principle
- When to use this skill
- Announce usage at start

## Prerequisites & Setup
- Google Sheets MCP server installation
- Authentication setup
- Verification steps

## Quick Reference
| Current State | Detected By | Next Phase |
|---------------|-------------|------------|
| No spreadsheet | No URL provided | Create spreadsheet |
| Empty spreadsheet | No sheets exist | Define options |
| Options exist | Metadata.options_defined=true | Define areas |
| ... | ... | ... |

## The Workflow
- Main checklist (copy to TodoWrite)
- Phase overview with links to detailed sections

## Phase Details
### Phase 0: MCP Server Setup
[Detailed instructions]

### Phase 1: State Detection
[State detection logic, menu presentation]

### Phase 2-7: [Each phase]
[Step-by-step instructions, prompts, operations]

## State Detection Logic
- How to read spreadsheet structure
- Metadata sheet parsing
- Checklist state inference
- Edge cases (corrupted state, version mismatches)

## Google Sheets Operations Reference
### Creating Sheets
[MCP commands and patterns]

### Writing Cells
[Batch write operations]

### Formatting Cells
[Color coding, merge cells, fonts]

### Adding Comments
[Cell comment syntax]

### Data Validation
[Dropdown, number ranges]

### Formulas
[SUMPRODUCT, AVERAGE, RANK, conditional formatting]

## Common Mistakes
- Missing MCP scopes
- Formula errors when adding options/considerations
- State corruption from manual sheet edits
- Incomplete scoring (forgetting cells)
- Weight normalization errors
- Area bias in formulas (wrong averaging)

## Anti-Patterns
- Manually editing Metadata sheet (use skill commands)
- Deleting area rows (corrupts formulas)
- Changing option column order (breaks Summary formulas)
- Skipping justifications (defeats audit purpose)

## Troubleshooting
[Common error messages and fixes]

## Example Workflow
[Complete walkthrough from start to finish]
```

**Estimated Length:** 450-550 lines with examples and detailed instructions.

## Integration with Other Skills

### writing-skills Skill
When implementing this design:
- Use `superpowers:writing-skills` to create the SKILL.md
- Test with subagents before deployment
- Follow RED-GREEN-REFACTOR for iterative refinement
- Create test scenarios: fresh analysis, resume mid-scoring, add option to complete analysis

### Elements-of-Style (if available)
- Apply clear, concise writing principles
- Use active voice for instructions
- Minimize jargon, define technical terms
- Use consistent terminology throughout

## Plugin Metadata

**File:** `plugins/options-analysis/.claude-plugin/plugin.json`

```json
{
  "name": "options-analysis",
  "description": "Structured multi-criteria decision analysis with weighted scoring - creates systematic Google Sheets for evaluating options",
  "version": "1.0.0",
  "author": {
    "name": "Ralph Bean",
    "email": "rbean@redhat.com"
  },
  "homepage": "https://github.com/ralphbean/claude-code-plugins",
  "repository": "https://github.com/ralphbean/claude-code-plugins",
  "license": "MIT",
  "keywords": ["decision-analysis", "mcda", "google-sheets", "scoring", "options"]
}
```

## Marketplace Entry

Update `.claude-plugin/marketplace.json`:

```json
{
  "name": "options-analysis",
  "description": "Structured multi-criteria decision analysis with weighted scoring spreadsheets",
  "version": "1.0.0",
  "source": "./plugins/options-analysis",
  "category": "productivity",
  "source_marketplace": "direct"
}
```

## Testing Strategy

### Manual Testing Scenarios
1. **Fresh Start:** New user, no spreadsheet → complete workflow → validate summary formulas
2. **Resume Scoring:** Stop mid-scoring, restart skill → verify correct state detection
3. **Add Option:** Complete analysis → add new option → verify formulas update correctly
4. **Add Consideration:** Complete analysis → add consideration to existing area → verify colors and formulas
5. **Modify Weights:** Change area weights → verify Summary recalculates correctly
6. **Edge Cases:**
   - Single consideration per area
   - Unequal area weights (80% one area, 20% distributed)
   - Maximum options (~10)
   - Maximum considerations (~50)

### Subagent Testing (via writing-skills)
1. Run subagent without skill → observe failures (confusion, incorrect formulas)
2. Run subagent with skill → verify it follows phases correctly
3. Iterate on unclear instructions based on subagent behavior

## Future Enhancements (Out of Scope for V1)

- Import options/considerations from CSV
- Multi-user collaboration tracking (who scored what)
- Sensitivity analysis (show impact of weight changes)
- Export to PDF report with charts
- Template library (pre-defined consideration sets for common decisions)
- Integration with other MCP servers (Jira for option tracking, LogSeq for decision log)

## Implementation Checklist

When building this skill:
- [ ] Set up git worktree (using `superpowers:using-git-worktrees`)
- [ ] Create plugin directory structure
- [ ] Write plugin.json metadata
- [ ] Write SKILL.md using `superpowers:writing-skills`
- [ ] Test with subagents
- [ ] Manually test all scenarios listed above
- [ ] Update marketplace.json
- [ ] Commit with signed-off GPG signature
- [ ] Create pull request

## References

- Google Sheets MCP Server: https://github.com/xing5/mcp-google-sheets
- Multi-Criteria Decision Analysis: https://en.wikipedia.org/wiki/Multiple-criteria_decision_analysis
- Existing skill pattern: `plugins/searching-slack-history/skills/searching-slack-history/SKILL.md`
