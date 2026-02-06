---
description: Iterative implementation and Codex review cycle with context-appropriate agents
argument-hint: [task description]
allowed-tools: Read, Write, Edit, Bash, Glob, Grep, Task, Bash(codex *)
---

# Implement & Review Cycle

Execute iterative development with continuous Codex-powered code review, automatically selecting the appropriate implementation agent based on the task context.

**Task**: $ARGUMENTS

## Agent Selection

Before starting, determine the most suitable implementation agent based on:

1. **Task description**: Identify explicit mentions of technologies, frameworks, or languages
2. **Target files/directories**: Examine file extensions, project configuration files, and directory structure
3. **Codebase analysis**: If ambiguous, explore the codebase to understand the primary technology stack

Select the agent that best matches the implementation context from the available specialized agents defined in `.claude/agents/`:

- `rust-engineer` — Rust projects
- `typescript-engineer` — TypeScript projects
- `python-engineer` — Python projects

If no specialized agent is a clear fit, use `general-purpose`.

The selected agent is referred to as `{impl-agent}` throughout this document.

## Validation Phase

Before proceeding, verify this workflow is properly configured:
- ✓ All phases (Design → Implement → Review → Iterate → Cleanup) are defined
- ✓ Phase 0 includes optional plan mode for design proposal (can be skipped if detailed documentation exists)
- ✓ Implementation agent has been selected based on context analysis
- ✓ Final phase includes code-simplifier for cleanup
- ✓ Quality gates are clearly documented
- ✓ Iteration limits are established
- ✓ Agent coordination is specified

## Workflow Overview

This command creates a continuous cycle of implementation and review:

0. **Analyze Context**: Determine the technology stack and select `{impl-agent}`
1. **Design (Plan Mode)** *(optional)*: Analyze requirements and propose implementation design
2. **Implement ({impl-agent})**: Build features with best practices appropriate to the stack
3. **Review (codex-code-reviewer)**: Comprehensive quality and security analysis
4. **Share Review Results**: Share Codex review results with the user
5. **Iterate**: Repeat until all quality standards are met
6. **Cleanup (code-simplifier)**: Simplify and refine code for clarity and maintainability
7. **Final Summary**: Provide suggested commit message (1 line) for version control

## Cycle Phases

### Phase 0: Design Proposal (Plan Mode)

**Focus**: Analyze requirements and propose implementation design

**Skip Conditions**:

This phase can be skipped if the provided task description already includes:
- Detailed implementation design with clear requirements
- API surface and type signatures
- Architecture decisions and rationale
- Clear scope and acceptance criteria
- Comprehensive task breakdown with specific deliverables

If the task documentation is sufficiently detailed (e.g., a well-defined spec in `docs/specs/{plan_name}/tasks/`), proceed directly to Phase 1 (Implementation). Otherwise, enter plan mode first.

Before writing any code, enter plan mode to:
- Thoroughly explore the codebase using Glob, Grep, and Read tools
- Understand existing patterns and architecture
- Analyze requirements for: $ARGUMENTS
- Identify relevant modules and dependencies
- Design API surface and type signatures
- Plan implementation approach
- Consider implications specific to the technology stack (safety, typing, performance, etc.)
- Identify potential challenges and trade-offs
- Evaluate multiple implementation strategies

**Deliverables**:
- Implementation design proposal
- API design with type signatures
- Architecture decisions and rationale
- Performance considerations
- Risk assessment
- User approval before proceeding

**Plan and Task Management**:

⚠️ **CRITICAL**: Plan mode creates files in `~/.claude/plans/` by default (OUTSIDE the project).

**Required Actions After Plan Mode**:
1. Read the plan file from `~/.claude/plans/` (created by EnterPlanMode)
2. **MUST** copy or recreate the content in `docs/specs/{plan_name}/spec.md`
3. Include metadata: Created date, Last Updated date, and Update Log
4. When creating detailed task breakdowns:
   - **MUST** store in `docs/specs/{plan_name}/tasks/` (NOT in global location)
   - Add task reference to spec.md update log: `- YYYY-MM-DD HH:MM:SS: Added task reference - tasks/task_001.md`
   - Add task completion to spec.md update log: `- YYYY-MM-DD HH:MM:SS: Task completed - tasks/task_001.md`
5. Use descriptive `{plan_name}` values (e.g., `fix_fluid_dex_2026_2_6`)
6. Ensure all specifications are version-controlled in the project repository

**Note**: Use the EnterPlanMode tool to formally propose the design. After approval and copying the plan to the project, proceed to Phase 1.

### Implementation Phase ({impl-agent})

**Agent**: `{impl-agent}` (selected during context analysis)
**Focus**: Build feature with best practices appropriate to the technology stack

Launch the selected implementation agent to:
- Write idiomatic code following the conventions of the target stack
- Apply best practices and patterns suited to the technology
- Add comprehensive documentation
- Write unit and integration tests

**Deliverables**:
- Working implementation
- Test suite
- Documentation with examples
- Performance benchmarks (if applicable)

### Review Phase (codex-code-reviewer)

**Agent**: `codex-code-reviewer` (see `.claude/agents/codex-code-reviewer.md` for full review criteria)
**Mode**: Full Review
**Focus**: Comprehensive quality and security analysis

Launch the `codex-code-reviewer` agent via the Task tool:

```
Task(subagent_type="codex-code-reviewer", prompt="Perform a Full Review (Mode 1) on the following changes for: $ARGUMENTS. Technology stack: {detected_stack}. Changed files: {changed_files}")
```

The `codex-code-reviewer` agent handles all review dimensions:
1. Code Quality Assessment
2. Security Review
3. Safety Analysis
4. Performance Analysis
5. Design Patterns & Architecture
6. Test Review
7. Documentation Review
8. Technical Debt Assessment

See `.claude/agents/codex-code-reviewer.md` for detailed review criteria, categories, and checklist.

### Share Review Results with User

⚠️ **CRITICAL**: This step MUST NOT be skipped under any circumstances.

Format the Codex review results and present them to the user:

```markdown
## Codex Review Results

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

Wait for the user to acknowledge the review results before proceeding to the next phase.

### Iteration Phase

**Agent**: `{impl-agent}` (same agent as Implementation Phase)
**Focus**: Address feedback and improve

Based on review feedback:
- Address critical issues immediately
- Prioritize high-priority improvements
- Consider medium-priority suggestions
- Document low-priority items for future work

**Deliverables**:
- Fixed critical issues
- Implemented improvements
- Updated tests
- Enhanced documentation

### Validation Phase (codex-code-reviewer)

**Agent**: `codex-code-reviewer` (see `.claude/agents/codex-code-reviewer.md`)
**Mode**: Validation Review
**Focus**: Verify improvements and approve or iterate

Launch the `codex-code-reviewer` agent in validation mode:

```
Task(subagent_type="codex-code-reviewer", prompt="Perform a Validation Review (Mode 2). Previous issues: {previous_issues}. Changed files: {changed_files}. Verify all issues are resolved and provide APPROVE or REQUEST CHANGES verdict.")
```

⚠️ **CRITICAL**: Follow-up review results MUST also be shared with the user.

**Deliverables**:
- Validation report
- Quality metrics
- Approval decision or iteration request

### Cleanup Phase

**Agent**: code-simplifier
**Focus**: Simplify and refine code for clarity, consistency, and maintainability

After all iterations are complete and code is approved, launch code-simplifier agent to:
- Simplify complex code structures while preserving functionality
- Improve code consistency across the implementation
- Enhance readability and maintainability
- Remove unnecessary abstractions or over-engineering
- Ensure naming conventions are clear and consistent
- Optimize code organization and structure
- Verify all tests still pass after simplification

**Deliverables**:
- Simplified and refined codebase
- Improved code clarity and readability
- Enhanced maintainability
- All tests still passing
- Documentation updated if needed
- Final code ready for merge
- **Suggested commit message (1 line)** for the implemented changes

**Important Notes**:
- This phase performs **cosmetic, non-functional changes only** — renaming for clarity, reducing boilerplate, removing dead code, improving formatting. It does NOT change logic, add features, or alter behavior.
- Because changes are strictly non-functional, an additional review cycle after this phase is not required.
- Code simplification should NOT compromise safety or functionality
- All quality gates must still pass after cleanup
- Performance characteristics should be maintained or improved
- This is a refinement step, not a rewrite
- Focus on recently modified code unless instructed otherwise

## Quality Gates

Quality gates are adapted based on the detected technology stack. Apply the checks relevant to the project's tooling and conventions.

### Gate 1: Build Succeeds
- ✓ Project builds/compiles without errors
- ✓ No warnings (or warnings addressed)
- ✓ Linter checks pass

### Gate 2: Tests Pass
- ✓ All tests passing
- ✓ Test coverage > 80%
- ✓ Integration tests pass

### Gate 3: Security & Safety
- ✓ No security vulnerabilities
- ✓ Dependency audit clean
- ✓ Input validation implemented
- ✓ No unsafe patterns in public API

### Gate 4: Performance
- ✓ No resource leaks
- ✓ Efficient algorithm usage
- ✓ No blocking calls in async contexts

### Gate 5: Documentation
- ✓ Public API fully documented
- ✓ Examples provided
- ✓ Architecture documented

## Usage Examples

```bash
# Feature implementation (agent auto-selected based on codebase)
/implement "Add async HTTP client with connection pooling"

# UI feature (agent auto-selected based on target files)
/implement "Add form validation to the signup components in frontend/src/"

# Data processing task
/implement "Add data pipeline for ETL processing in scripts/"

# Infrastructure change
/implement "Add WebSocket support for real-time order updates"
```

## Iteration Control

**Maximum Iterations**: 3 cycles

If quality gates are not passed after 3 iterations, escalate for architectural review.

**Success Criteria**:
- All 5 quality gates passed
- No critical or high-priority issues remaining
- Code quality score > 90%
- Performance targets met

## Agent Coordination

Use the Task tool to invoke agents in the following sequence:

0. **Analyze context** and select `{impl-agent}` based on the task and codebase
1. Enter plan mode to design and propose implementation (wait for user approval) - **Skip if task documentation is already detailed**
2. Launch `{impl-agent}` with implementation task
3. Wait for completion, then launch `codex-code-reviewer` (Mode 1: Full Review)
4. **Share review results with the user and wait for acknowledgment**
5. If issues found, launch `{impl-agent}` for iteration
6. Wait for completion, then launch `codex-code-reviewer` (Mode 2: Validation Review)
7. **Share follow-up review results with the user and wait for acknowledgment**
8. Repeat steps 5-7 until approved or max iterations reached
9. Wait for final approval, then launch `code-simplifier:code-simplifier` for cleanup
10. Provide final summary with a **suggested commit message (1 line)** describing the implemented changes

## Benefits

1. **Technology Agnostic**: Automatically selects the best implementation agent based on the task context
2. **Continuous Quality**: External review perspective from Codex ensures high standards
3. **Iterative Improvement**: Progressive refinement of code
4. **Fast Feedback**: Quick identification of issues
5. **Cross-Model Review**: Multi-perspective analysis using Claude (implementation) + Codex (review)
6. **Safety First**: Context-appropriate safety checks at each iteration
7. **Code Clarity**: Final cleanup ensures maintainable, readable code
8. **Transparency**: Review results are always shared with the user for informed decision-making

## Best Practices

1. **Start Small**: Begin with small, focused changes
2. **Review Early**: Don't wait for complete features
3. **Iterate Quickly**: Short cycles are more effective
4. **Learn Continuously**: Use reviews as learning opportunities
5. **Document Decisions**: Record architectural choices
6. **Measure Progress**: Track metrics over time
7. **Automate Checks**: Use CI/CD for consistent quality
8. **Embrace Feedback**: View reviews as improvement opportunities
9. **Simplify Last**: Run code-simplifier after all iterations to ensure clean, maintainable code
10. **Finish with Commit Message**: Provide a concise, 1-line commit message summarizing the implementation for easy version control
