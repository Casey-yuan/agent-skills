---
name: "code-analyzer"
description: "Analyze error logs, parse API interfaces, and diagnose interface issues. Invoke when user provides error logs followed by '分析', provides interface address followed by '解释', or provides interface address followed by '现象'."
---

# Code Analyzer

This skill provides comprehensive code analysis capabilities including error log analysis, API interface parsing, and interface issue diagnosis.

## Features

### 1. Error Log Analysis
- Receives error log text from users
- Automatically parses stack trace information to pinpoint the exact interface name causing the error
- Identifies and marks specific error locations including file paths, function names, and line numbers
- Provides concrete, actionable modification suggestions based on error type and context
- Presents detailed modification plan with before/after code comparisons for user review
- Adds clear comments in modified code explaining the root cause and solution
- Only applies modifications after explicit user approval

### 2. API Interface Parsing & Logic Understanding
- Receives API endpoint addresses from users
- Quickly locates the specific implementation location of the interface in the project code
- Detailed analysis of interface call logic including:
  * Request parameter processing flow
  * Business logic execution steps
  * Data processing and transformation processes
  * Response result generation mechanism
  * Exception handling strategies
- Presents interface logic in a clear, structured manner to help users quickly understand interface functionality and implementation details
- Identifies potential improvement areas with suggested modifications
- Provides modification proposals with code comments explaining the improvement rationale
- Only applies changes after explicit user approval

### 3. Interface Issue Diagnosis & Fix Suggestions
- Receives interface address and problem description from users
- Analyzes potential problem areas in the interface implementation
- Identifies suspicious code patterns that may cause the reported issue
- Checks common issues including:
  * Parameter validation and sanitization
  * Null/undefined handling
  * Database query performance
  * Error handling completeness
  * Edge case coverage
- Provides targeted modification suggestions with code examples
- Offers multiple solution options when applicable
- Presents detailed fix plan with clear modification rationale for user review
- Adds comprehensive comments in fixed code explaining why the change was made
- Only implements fixes after user explicit confirmation

## Usage Examples

**Error Log Analysis:**
```
User: "[完整的错误日志内容] 分析"
Agent: Invokes code-analyzer skill
```

**Interface Parsing:**
```
User: "/api/users 解释"
Agent: Invokes code-analyzer skill
```

**Interface Issue Diagnosis:**
```
User: "/api/orders 现象：当用户ID为null时返回500错误"
Agent: Invokes code-analyzer skill
```

## Interaction Flow

1. **Error Analysis:**
   - User provides error log followed by "分析"
   - Skill parses stack trace and identifies error location
   - Skill provides detailed modification plan with before/after code comparisons
   - User reviews the modification plan
   - User confirms approval
   - Skill applies modifications with explanatory comments describing root cause and solution
   - Skill only applies modifications after explicit user approval

2. **Interface Analysis:**
   - User provides interface address followed by "解释"
   - Skill searches for interface implementation
   - Skill analyzes call flow and logic
   - Skill presents structured analysis results with potential improvement suggestions
   - Skill provides modification proposals with code comments explaining improvement rationale
   - User reviews the proposals
   - Skill applies changes with comments explaining why the improvement was made
   - Skill only applies changes after explicit user approval

3. **Interface Issue Diagnosis:**
   - User provides interface address followed by "现象" and problem description
   - Skill locates interface implementation
   - Skill analyzes code for potential issues related to the reported problem
   - Skill identifies suspicious code patterns and root causes
   - Skill provides detailed fix plan with code examples and multiple solution options
   - User reviews the fix plan
   - User confirms approval
   - Skill implements fixes with comprehensive comments explaining the fix rationale
   - Skill only implements fixes after explicit user confirmation

## Code Comment Standards

When modifying code, the following comment standards will be applied:

- **Root Cause Comment**: Explains why the issue occurred (e.g., "// Fix: Null pointer exception caused by missing user ID validation")
- **Solution Comment**: Describes what the fix achieves (e.g., "// Added null check to prevent NPE when userId is null")
- **Context Comment**: Provides background information if needed (e.g., "// Related to issue: /api/orders returns 500 when user ID is null")
- **Rationale Comment**: Explains why this particular solution was chosen over alternatives (e.g., "// Using early return pattern for better readability")
