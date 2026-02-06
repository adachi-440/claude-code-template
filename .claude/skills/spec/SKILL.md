---
description: Spec Command
argument-hint: [feature_name]
allowed-tools: Read, Write, Edit, Glob, Grep, Task, AskUserQuestion
---

# Spec Command

Create a new feature specification with interactive prompts.

**Task**: Create specification for `$ARGUMENTS`

## Validation Phase

Before proceeding, verify this workflow is properly configured:
- ✓ Template file exists at `.claude/skills/spec/spec-template.md`
- ✓ Target directory `docs/specs/` is accessible
- ✓ Feature name follows kebab-case convention
- ✓ Interactive prompts are ready for information gathering

## Workflow Overview

This command creates a feature specification document following the project's standard format:

1. **Validate Input**: Verify feature name and check for existing specs
2. **Gather Information**: Interactive prompts for specification details
3. **Generate Spec**: Create specification document from template
4. **Setup Structure**: Create tasks directory for future task breakdowns
5. **Codex Review**: Request specification review via `/ask-codex` skill
6. **Share Results**: Share Codex review results with the user
7. **Summary**: Output created files and next steps

## Phases

### Phase 1: Input Validation

**Focus**: Validate feature name and environment

**Actions**:
1. Parse feature name from `$ARGUMENTS`
2. Convert to kebab-case if needed (e.g., `userAuth` → `user-auth`)
3. Check if `docs/specs/{feature_name}/` already exists
4. If exists, warn user and confirm before overwriting

**Validation Checks**:
- [ ] Feature name is provided
- [ ] Feature name follows naming conventions
- [ ] No conflicting existing specification

### Phase 2: Information Gathering

**Focus**: Collect specification details through codebase exploration and interactive prompts

**Exploration Phase**:
Using the Task tool with `subagent_type=Explore`, gather context about the feature:

1. **Codebase Analysis**
   - Search for existing implementations related to the feature
   - Identify relevant files, modules, and patterns
   - Find similar features for reference
   - Understand current architecture and conventions

2. **Dependency Discovery**
   - Identify potential dependencies and integrations
   - Find related configuration files
   - Discover existing utilities or helpers that could be reused

**Interactive Prompts**:
After exploration, use AskUserQuestion tool to gather the following information:

1. **Overview** (Required)
   - Question: "Please provide a brief overview of the feature (1-3 sentences)"
   - Purpose: Summary of what the feature does

2. **Goals** (Required)
   - Question: "What are the main goals of this feature? (List 2-5 goals)"
   - Purpose: Clear success criteria

3. **Background/Context** (Optional)
   - Question: "Is there any background context or motivation for this feature?"
   - Purpose: Why this feature is needed

4. **Non-goals** (Optional)
   - Question: "Are there any explicit non-goals or out-of-scope items?"
   - Purpose: Define boundaries

5. **Technical Considerations** (Optional)
   - Question: "Are there any technical constraints or dependencies?"
   - Purpose: Implementation context
   - Note: Pre-populate with findings from Explore phase

### Phase 3: Specification Generation

**Focus**: Create the specification document

**Actions**:
1. Create directory: `docs/specs/{feature_name}/`
2. Select the template based on language:
   - If the user explicitly specifies a language (e.g., "in English"), use that.
   - Otherwise, match the language the user has been using in the conversation.
   - Japanese → `.claude/skills/spec/spec-template.md`
   - English → `.claude/skills/spec/spec-template.en.md`
3. Fill in template with collected information:
   - Replace `{TITLE}` with formatted feature name
   - Replace `{CREATED_DATE}` with current timestamp (YYYY-MM-DD HH:MM:SS)
   - Replace `{OVERVIEW}` with user-provided overview
   - Replace `{GOALS}` with formatted goals list
   - Replace `{BACKGROUND}` with context (if provided)
   - Replace `{NON_GOALS}` with non-goals list (if provided)
   - Replace `{TECHNICAL_CONSIDERATIONS}` with constraints (if provided)
4. Write to `docs/specs/{feature_name}/spec.md`

### Phase 4: Structure Setup

**Focus**: Prepare for task breakdown

**Actions**:
1. Create empty tasks directory: `docs/specs/{feature_name}/tasks/`
2. This directory will be used by `/task` command

### Phase 5: Codex Review

**Agent**: `codex-code-reviewer` (see `.claude/agents/codex-code-reviewer.md`)
**Mode**: Spec Review
**Focus**: Request a review of the generated specification

Launch the `codex-code-reviewer` agent via the Task tool:

```
Task(subagent_type="codex-code-reviewer", prompt="Perform a Spec Review (Mode 3) on docs/specs/{feature_name}/spec.md. Check design validity, completeness, risks, improvement suggestions, and consistency with CoW Protocol architecture.")
```

### Phase 6: Share Review Results with User

**Focus**: Share the Codex review results with the user

⚠️ **CRITICAL**: This phase MUST NOT be skipped under any circumstances. The Codex review results MUST be displayed to the user so they can review and acknowledge the feedback.

**Actions**:
1. Format the Codex review results and present them to the user:

```markdown
## Codex Review Results for {feature_name}

### Review Summary
{Summary of the Codex review}

### Issues Found
#### Critical
- {Critical issues}

#### High
- {High priority issues}

#### Medium
- {Medium priority issues}

#### Suggestions
- {Improvement suggestions}

### Verdict
{Overall assessment and recommendations}
```

2. Ask the user whether to revise the specification based on the review feedback
3. If revisions are needed, update the specification and record the review response in the Update Log

### Phase 7: Summary

**Focus**: Confirm completion and provide next steps

**Output**:
```
Specification created successfully!

Files created:
- docs/specs/{feature_name}/spec.md
- docs/specs/{feature_name}/tasks/ (directory)

Codex Review:
- Review completed, see results above

Next steps:
1. Review the specification: docs/specs/{feature_name}/spec.md
2. Add implementation plan details if needed
3. Generate task files with: /task {feature_name}
4. Implement with: /implement "Implement {feature_name} based on docs/specs/{feature_name}/spec.md"
```

## Template Location

**Template File**: `.claude/skills/spec/spec-template.md`

The template follows the project's established specification format with:
- Metadata header (Created, Last Updated, Update Log)
- Structured sections for overview, goals, background, etc.
- Implementation plan section for task breakdown
- Validation checklist

## Usage Examples

```bash
# Create a new feature specification
/spec fluid-dex-integration

# Create specification for new solver
/spec bebop-solver

# Create specification for architecture change
/spec auction-pipeline-refactor
```

## Naming Conventions

**Feature Name Format**:
- Use kebab-case: `fluid-dex-integration`, `bebop-solver`, `auction-pipeline`
- Be descriptive but concise
- Avoid abbreviations unless widely understood

**Generated Directory Structure**:
```
docs/specs/{feature_name}/
├── spec.md              # Main specification document
└── tasks/               # Task breakdown directory (empty initially)
    ├── task_001_xxx.md  # (Created by /task command)
    └── task_002_xxx.md
```

## Error Handling

### Existing Specification
If a specification already exists:
- Warn the user
- Ask for confirmation to overwrite or choose different name
- Never silently overwrite existing work

### Missing Template
If template file is missing:
- Create default template structure
- Log warning about missing template
- Continue with basic specification format

### Invalid Feature Name
If feature name is invalid:
- Suggest corrections (remove special characters, convert to kebab-case)
- Ask user to confirm corrected name

## Integration with Other Commands

**Workflow Integration**:
1. `/spec {feature_name}` - Create specification (this command)
2. `/task {feature_name}` - Generate task breakdown from spec
3. `/implement "Implement based on spec"` - Implement with review cycle

## Best Practices

1. **Be Specific**: Provide detailed information in prompts
2. **Define Boundaries**: Clearly state non-goals to prevent scope creep
3. **Consider Dependencies**: Note any technical constraints early
4. **Review Before Tasks**: Ensure spec is complete before generating tasks
5. **Update Log**: Keep the update log current when spec changes

## Deliverables

After running this command, you will have:
- A well-structured specification document
- Clear goals and boundaries defined
- A directory ready for task breakdown
- Foundation for implementation planning
