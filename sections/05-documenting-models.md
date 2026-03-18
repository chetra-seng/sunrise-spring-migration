---
layout: center
---
# Documenting Models

## @Schema on DTOs

---

# Why Document DTOs?

Your request and response DTOs appear in Swagger UI as schemas. By default, they show field names and types — but no descriptions.

<div class="grid grid-cols-2 gap-6 mt-4">

<div class="border rounded-lg p-4 border-red-300 bg-red-50 dark:bg-red-950">

**Without @Schema:**
```json
{
  "title": "string",
  "description": "string"
}
```
What's the max length for `title`? Is `description` required?

</div>

<div class="border rounded-lg p-4 border-green-300 bg-green-50 dark:bg-green-950">

**With @Schema:**
```json
{
  "title": "Implement login page",
  "description": "Create login page with email and password fields"
}
```
Example values make the contract clear instantly.

</div>

</div>

---

# @Schema on the Class

Put `@Schema` on the DTO class to give it a name and description:

```java
@Schema(
    name = "TaskRequest",
    description = "Data required to create or update a task"
)
@Data
@NoArgsConstructor
@AllArgsConstructor
public class TaskRequest {

    private String title;
    private String description;
}
```

---
zoom: 0.85
---

# @Schema on Fields — TaskRequest

Put `@Schema` on individual fields to add descriptions, examples, and constraints:

```java
@Schema(name = "TaskRequest", description = "Data required to create or update a task")
@Data
@NoArgsConstructor
@AllArgsConstructor
public class TaskRequest {

    @Schema(description = "Task title (max 100 characters)", example = "Implement login page")
    private String title;

    @Schema(description = "Detailed task description", example = "Create login page with email and password fields")
    private String description;
}
```

---
zoom: 0.85
---

# @Schema on Response DTOs

Document your response DTOs too — consumers need to know what comes back:

```java
@Schema(name = "TaskResponse", description = "Full task data returned by the API")
@Data
@AllArgsConstructor
@NoArgsConstructor
public class TaskResponse {

    @Schema(description = "Auto-generated task ID", example = "42")
    private Long id;

    @Schema(description = "Task title", example = "Implement login page")
    private String title;

    @Schema(description = "Task description", example = "Create login page with email and password fields")
    private String description;

    @Schema(description = "Whether the task has been completed", example = "false")
    private Boolean completed;

    @Schema(description = "Timestamp when the task was created", example = "2026-03-17T10:30:00")
    private LocalDateTime createdAt;

    @Schema(description = "Name of the project this task belongs to", example = "Task Management System")
    private String projectName;

    @Schema(description = "ID of the project", example = "1")
    private Long projectId;
}
```

---

# API Design Best Practices

A few rules that make your documentation (and API) genuinely useful:

<v-clicks>

**Use meaningful example values** — `"Implement login page"` tells more than `"string"`

**Document error responses** — a 404 with a schema is more helpful than a mystery 404

**Mark required vs optional clearly** — `requiredMode = Schema.RequiredMode.REQUIRED`

**Document enum allowable values** — if a field is an enum, use `allowableValues = {"A","B","C"}` to list them

**Version your API** — `/api/v1/tasks` is clearer than `/api/tasks` when you make breaking changes

</v-clicks>

---

# The Result

After annotating your controllers and DTOs, Swagger UI will show:

<v-clicks>

- **Tasks** tag with a description at the top
- All 7 endpoints (GET all, GET by id, POST, PUT, DELETE, PATCH complete, GET filter)
- Each endpoint with summary, description, and example request/response
- Request body schema with field names, types, descriptions, and examples
- Response schemas for 200, 201, 400, 404 cases
- A "Try it out" form pre-filled with your example values

</v-clicks>

<v-click>

This is what you'll build in the next lab.

</v-click>

<!--
Students often don't realize how much documentation they're generating until they see the full Swagger UI. Good moment to demo a fully annotated controller.
-->
