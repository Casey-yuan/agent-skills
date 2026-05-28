---
name: "table-structure-generator"
description: "Generates database table structure documentation in Markdown from code entities (JPA/MyBatis annotations). Invoke when user asks to generate table docs, export DDL to MD, or document database schema from code."
---

# Table Structure Generator

This skill analyzes Java code (entity classes, JPA annotations, MyBatis mappings, etc.) and generates database table structure documentation in Markdown format.

## When to Invoke

- User asks to generate table structure documentation from code
- User wants to export database schema as Markdown
- User asks to document entity classes as table definitions
- User requests DDL or table structure output in MD format

## Steps

1. **Scan for Entity Classes**: Search the codebase for entity/model classes that contain database mapping annotations such as:
   - JPA: `@Entity`, `@Table`, `@Column`, `@Id`, `@GeneratedValue`
   - MyBatis-Plus: `@TableName`, `@TableId`, `@TableField`
   - MyBatis: XML mapper files with table definitions
   - Hibernate: `@Entity`, `@Table` with various column annotations

2. **Extract Table Metadata**: For each entity class, extract:
   - Table name (from `@Table(name="...")` or `@TableName("...")`)
   - Column name, type, length, nullable, default value
   - Primary key information
   - Index information
   - Comments/descriptions (from `@Comment` or field Javadoc)
   - Relationships (foreign keys, join tables)
   - **Column References**: For each column, infer if it references another table's column through code analysis (NOT from annotations). Infer from:
     - Field naming conventions: `userId` → search for a `User` entity class → references `user.id`, `deptId` → search for `Dept`/`Department` entity → references corresponding table's id
     - Naming pattern inference: if a column name ends with `_id` or `Id`, extract the prefix, search the codebase for an entity class matching that name, and if found, mark as referencing that entity's primary key
     - Service/Controller code: look for patterns like `getById(userId)` or `userService.getById(xxx)` which imply the field references another entity
     - SQL/XML mapper files: look for JOIN conditions like `t.user_id = u.id` or WHERE clauses referencing other tables
     - Code comments or Javadoc containing reference hints (e.g., `// 关联用户表ID`)
     - DO NOT rely on JPA relationship annotations (`@ManyToOne`, `@JoinColumn`, etc.) or `@ForeignKey` — always infer from actual code usage and naming patterns

3. **Resolve Enum Fields**: For any field whose type is a Java enum, determine the enum values by combining enum class definition analysis with usage-based discovery:

   **a) From Enum Class Definition:**
   - Search the codebase for the enum class definition
   - Extract each enum constant name
   - If the enum has custom fields (e.g., `code`, `desc`, `label`), extract those values as well
   - Common enum patterns to handle:
     - Simple enum: `ACTIVE, INACTIVE, DELETED`
     - Enum with code/desc: `ACTIVE(1, "启用"), INACTIVE(0, "停用")`
     - Enum with `@EnumValue` (MyBatis-Plus): extract the annotated field value
     - Enum implementing custom interface: extract the interface method return values

   **b) From Usage Analysis (Primary Method):**
   - Search the codebase for all references to the enum field across the project (service classes, controllers, mappers, XML files, etc.)
   - Analyze how the enum field is used in practice to discover actual enum values:
     - Direct enum constant references: `StatusEnum.ACTIVE`, `StatusEnum.INACTIVE`
     - Comparisons and assignments: `setStatus(StatusEnum.ACTIVE)`, `if (status == StatusEnum.DELETED)`
     - Switch/case statements: `case ACTIVE:`, `case INACTIVE:`
     - Collections and arrays: `Arrays.asList(StatusEnum.ACTIVE, StatusEnum.INACTIVE)`
     - Annotations: `@EnumValue`, validation constraints referencing enum values
     - SQL/XML mapper files: hardcoded values like `status = 1` or `status IN (0, 1, 2)` that correspond to enum ordinals/codes
     - Constants and config files: property files or YAML that reference enum values
   - Merge discovered values from usage analysis with the enum class definition
   - If the enum class cannot be found, rely entirely on usage analysis to infer possible values
   - If no enum class and no usage found, just note the field type name in the description

   **c) Output Format:**
   - Format enum values into the Description column, e.g.: `状态 (枚举: 1-启用, 0-停用, 2-删除)`
   - If values are discovered only from usage (no enum class found), note as: `状态 (推断枚举: 1-启用, 0-停用)`

4. **Generate Markdown Output**: Produce a well-structured Markdown document with:
   - Table overview (table name, description)
   - Column definition table with headers: | Column Name | Type | Length | Nullable | Default | Reference | Description |
   - Primary key indicator
   - Index information (if available)

5. **Output Format**: The generated Markdown should follow this template:

```markdown
# Database Table Structure

## Table: `table_name`

**Description**: Table description here

| Column Name | Type | Length | Nullable | Default | Reference | Description |
|-------------|------|--------|----------|---------|-----------|-------------|
| `id` | BIGINT | 20 | NO | - | - | Primary key |
| `name` | VARCHAR | 255 | YES | NULL | - | User name |
| `status` | INT | 11 | NO | 1 | - | 状态 (枚举: 1-启用, 0-停用, 2-删除) |
| `user_id` | BIGINT | 20 | NO | - | `user.id` | 关联用户ID |
| `dept_id` | BIGINT | 20 | YES | NULL | `department.id` | 关联部门ID |

**Primary Key**: `id`

**Indexes**:
- `idx_name` on `name`
```

6. **Save or Output**: Output the generated Markdown content. If the user specifies a file path, save it to that location.

## Important Notes

- **MANDATORY: Complete Output** — You MUST output the full table structure for ALL discovered tables. NEVER truncate, summarize, or omit any table with phrases like "due to length limitations", "see full document for remaining tables", "以下模块的详细表结构请参见完整文档", or any similar shortcuts. Every single table must be fully documented with all its columns, indexes, and foreign keys. If the output is large, output it in full without any abbreviation.
- If no annotations are found, try to infer table structure from class names and field names using naming conventions (camelCase to snake_case conversion)
- Support multiple entity classes in a single run
- Group tables by module/package if applicable
- Include Chinese comments if present in the source code
