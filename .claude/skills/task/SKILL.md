---
description: Task Command
argument-hint: [feature_name or spec_path]
allowed-tools: Read, Write, Edit, Glob, Grep, AskUserQuestion
---

# Task Command

Generate individual task files from a feature specification.

**Task**: Generate task breakdown for `$ARGUMENTS`

## Validation Phase

Before proceeding, verify this workflow is properly configured:
- ✓ Specification file exists at `docs/specs/{feature_name}/spec.md`
- ✓ Template file exists at `.claude/skills/task/task-template.md`
- ✓ Specification contains implementation plan or task breakdown section
- ✓ Tasks directory is ready for file creation

## Workflow Overview

This command automates the generation of task files from a feature specification:

1. **Resolve Spec**: Find and read the specification file
2. **Parse Tasks**: Extract task breakdown from implementation plan
3. **Interactive Refinement**: Refine each task with user input
4. **Generate Files**: Create individual task documents
5. **Update Spec**: Add task references to spec.md update log
6. **Summary**: Output created files and next steps

## Phases

### Phase 1: Specification Resolution

**Focus**: Locate and validate the specification file

**Input Handling**:
The `$ARGUMENTS` can be:
- Feature name only: `fluid-dex-integration` → resolves to `docs/specs/fluid-dex-integration/spec.md`
- Full path: `docs/specs/fluid-dex-integration/spec.md`
- Relative path: `specs/fluid-dex-integration/spec.md`

**Actions**:
1. Parse input from `$ARGUMENTS`
2. Resolve to full specification path
3. Verify file exists
4. Read specification content
5. Extract feature name for directory operations

**Validation Checks**:
- [ ] Specification file exists
- [ ] Specification is readable
- [ ] Tasks directory exists or can be created

### Phase 2: Task Extraction

**Focus**: Parse tasks from specification content

**Scan for Task Information**:
Look for task breakdown in these sections:
1. "Implementation Plan" section
2. "Task Breakdown" subsection
3. "Phases" section with numbered tasks
4. Bulleted or numbered lists under "Tasks" heading

**Extraction Rules**:
- Each main bullet point or numbered item = one task
- Sub-items become task details
- Phase headers provide grouping context
- Estimated times extracted if present

**Example Parsing**:
```markdown
## Implementation Plan

### Phase 1: Data Layer (2h)
1. Create data structures
2. Add serialization
3. Implement storage layer

### Phase 2: API Implementation (4h)
1. Define endpoints
2. Add validation
3. Implement handlers
```

Results in 6 tasks:
- `task_001_create_data_structures.md`
- `task_002_add_serialization.md`
- `task_003_implement_storage.md`
- `task_004_define_endpoints.md`
- `task_005_add_validation.md`
- `task_006_implement_handlers.md`

### Phase 3: Interactive Refinement

**Focus**: Refine each task with user input (optional)

For each extracted task, optionally ask:

1. **Task Description**
   - Question: "Refine the description for task: {task_name}"
   - Default: Use extracted description

2. **Files to Modify**
   - Question: "What files will this task modify?"
   - Purpose: Document scope

3. **Implementation Notes**
   - Question: "Any specific implementation notes?"
   - Purpose: Capture important details

4. **Dependencies**
   - Question: "Does this task depend on other tasks?"
   - Purpose: Define execution order

**Skip Option**:
If user wants to skip refinement:
- Use extracted information directly
- Generate basic task files
- User can edit files manually later

### Phase 4: Task File Generation

**Focus**: Create individual task documents

**File Naming Convention**:
```
task_{NNN}_{task_name_slug}.md
```
Where:
- `{NNN}`: Zero-padded task number (001, 002, 003...)
- `{task_name_slug}`: Kebab-case task name

**Template Processing**:
1. Select the template based on language:
   - If the user explicitly specifies a language (e.g., "in English"), use that.
   - Otherwise, match the language the user has been using in the conversation.
   - Japanese → `.claude/skills/task/task-template.md`
   - English → `.claude/skills/task/task-template.en.md`
2. Replace placeholders:
   - `{TASK_TITLE}`: Formatted task name
   - `{CREATED_DATE}`: Current timestamp (YYYY-MM-DD HH:MM:SS)
   - `{PARENT_SPEC}`: Path to parent spec (relative: `../spec.md`)
   - `{DESCRIPTION}`: Task description
   - `{FILES_TO_MODIFY}`: List of files
   - `{IMPLEMENTATION_NOTES}`: Any additional notes
   - `{DEPENDENCIES}`: Task dependencies
   - `{ESTIMATED_TIME}`: Time estimate (if available)
3. Write to `docs/specs/{feature_name}/tasks/task_{NNN}_{slug}.md`

### Phase 5: Specification Update

**Focus**: Update the parent specification

**Actions**:
1. Open `docs/specs/{feature_name}/spec.md`
2. Update `Last Updated` timestamp
3. Add entries to Update Log for each created task:
   ```markdown
   - YYYY-MM-DD HH:MM:SS: Added task reference - tasks/task_001_xxx.md
   - YYYY-MM-DD HH:MM:SS: Added task reference - tasks/task_002_xxx.md
   ```

### Phase 6: Summary

**Focus**: Report completion and next steps

**Output**:
```
Task files generated successfully!

Created tasks:
- docs/specs/{feature_name}/tasks/task_001_xxx.md
- docs/specs/{feature_name}/tasks/task_002_xxx.md
- docs/specs/{feature_name}/tasks/task_003_xxx.md

Updated:
- docs/specs/{feature_name}/spec.md (added task references to update log)

Next steps:
1. Review generated task files
2. Adjust priorities or dependencies if needed
3. Begin implementation with:
   /implement "Implement task_001 based on docs/specs/{feature_name}/tasks/task_001_xxx.md"
```

## Template Location

**Template File**: `.claude/skills/task/task-template.md`

The template follows the project's established task document format with:
- Metadata header (Created, Last Updated, Parent Spec)
- Update Log for tracking changes
- Structured sections for description, files, notes
- Success criteria checklist

## Usage Examples

```bash
# Generate tasks using feature name
/task fluid-dex-integration

# Generate tasks using spec file path
/task docs/specs/bebop-solver/spec.md

# Generate tasks for specific feature
/task auction-pipeline-refactor
```

## Task Document Structure

Each generated task file follows this structure:

```markdown
# {Task Title}

**Created**: YYYY-MM-DD HH:MM:SS
**Last Updated**: YYYY-MM-DD HH:MM:SS
**Parent Spec**: ../spec.md

## Update Log
- YYYY-MM-DD HH:MM:SS: Initial creation

---

## Overview
{Description of what this task accomplishes}

## Files to Modify
- `crates/path/to/file.rs`
- `crates/path/to/another_file.rs`

## Implementation Details
{Detailed implementation notes}

## Dependencies
- Requires: task_001_xxx.md (if any)

## Success Criteria
- [ ] Criterion 1
- [ ] Criterion 2
```

## Error Handling

### Missing Specification
If specification file not found:
- List available specifications in `docs/specs/`
- Ask user to confirm or create new spec with `/spec`

### No Tasks Found
If no task breakdown in specification:
- Warn user that spec lacks implementation plan
- Ask if user wants to add tasks manually
- Suggest adding "Implementation Plan" section to spec

### Existing Task Files
If task files already exist:
- Warn user about existing files
- Ask for confirmation to regenerate or skip existing

### Invalid Task Names
If extracted task names are unclear:
- Ask user to provide clear task name
- Suggest naming conventions

## Integration with Other Commands

**Workflow Integration**:
1. `/spec {feature_name}` - Create specification first
2. `/task {feature_name}` - Generate task breakdown (this command)
3. `/implement "Implement task"` - Implement individual tasks

**Task Lifecycle**:
1. Task created by `/task` command
2. Implementation with `/implement`
3. Update task's update log with completion
4. Update parent spec's update log with task completion

## Best Practices

1. **Complete Spec First**: Ensure specification has clear implementation plan
2. **Review Before Generating**: Verify task breakdown makes sense
3. **Track Dependencies**: Note which tasks depend on others
4. **Update Logs**: Keep update logs current when tasks change
5. **One Task at a Time**: Implement tasks sequentially for best results

## Deliverables

After running this command, you will have:
- Individual task files for each implementation step
- Clear scope and criteria for each task
- Updated specification with task references
- Foundation for systematic implementation
