---
layout: center
---
# Lab: Document the Task Flow API

## Hands-On Practice — Swagger

<!--
Give students 20 minutes. Walk around and help. Most common issue: wrong import for @ApiResponse.
-->

---

# What You're Doing

Your Task Flow API runs but has no documentation. By the end of this lab, every endpoint will be fully documented in Swagger UI.

<v-clicks>

**Four steps:**
1. Add `springdoc-openapi` dependency
2. Create `OpenApiConfig` bean
3. Annotate `TaskController` with `@Tag`, `@Operation`, `@ApiResponse`
4. Annotate `TaskRequest` and `TaskResponse` with `@Schema`

**Goal:** Open `/swagger-ui.html` and see a fully documented Task Flow API with working "Try it out."

</v-clicks>

---

# Step 1 — Add the Dependency

Open `pom.xml` and add inside `<dependencies>`:

```xml
<!-- Swagger / OpenAPI documentation -->
<dependency>
    <groupId>org.springdoc</groupId>
    <artifactId>springdoc-openapi-starter-webmvc-ui</artifactId>
    <version>3.0.2</version>
</dependency>
```

Then run:

```bash
./mvnw dependency:resolve
```

Or just restart your IDE — it will download the dependency automatically.

---

# Step 2 — Create OpenApiConfig.java

Create `src/main/java/com/chetraseng/sunrise_task_flow_api/config/OpenApiConfig.java`:

```java
package com.chetraseng.sunrise_task_flow_api.config;

import io.swagger.v3.oas.models.OpenAPI;
import io.swagger.v3.oas.models.info.Contact;
import io.swagger.v3.oas.models.info.Info;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class OpenApiConfig {

    @Bean
    public OpenAPI taskFlowApiOpenAPI() {
        return new OpenAPI()
            .info(new Info()
                .title("Task Flow API")
                .description("REST API for managing tasks, projects, and users")
                .version("v1.0")
                .contact(new Contact()
                    .name("Sunrise Team")));
    }
}
```

Start the app and open `http://localhost:8888/swagger-ui.html` — you should see your API.

---
zoom: 0.85
---

# Step 3 — Annotate TaskController (class + findAll)

Open `TaskController.java` and add these annotations. Start with the class-level `@Tag`:

```java
@Tag(name = "Tasks", description = "Endpoints for managing tasks")
@RestController
@RequestMapping("/api/tasks")
@RequiredArgsConstructor
public class TaskController {

    private final TaskService taskService;

    @Operation(summary = "Get all tasks",
               description = "Returns all tasks. Optionally filter by ?completed=true/false. Use /filter for paginated results.")
    @GetMapping
    public List<TaskResponse> getAllTask(
            @Parameter(description = "Filter by completion status", example = "false")
            @RequestParam(required = false) Boolean completed) {
        return taskService.findAll();
    }
    // ... continue for other endpoints
}
```

---

# Step 3 — Annotate findById

`findById` is a good place to practice `@ApiResponses` — it can return 200 or 404:

```java
@Operation(summary = "Get task by ID")
@ApiResponses({
    @ApiResponse(responseCode = "200", description = "Task found",
        content = @Content(schema = @Schema(implementation = TaskResponse.class))),
    @ApiResponse(responseCode = "404", description = "Task not found",
        content = @Content(schema = @Schema(implementation = ErrorResponse.class)))
})
@GetMapping("/{id}")
public TaskResponse getTaskById(
        @Parameter(description = "Task ID", example = "1")
        @PathVariable Long id) {
    return taskService.findById(id);
}
```

---

# Step 3 — Annotate create, update & delete

These three share a simple pattern — one `@Operation` summary, one `@ApiResponse` for the success code:

```java
@Operation(summary = "Create a new task")
@ApiResponse(responseCode = "201", description = "Task created")
@PostMapping
public ResponseEntity<TaskResponse> createTask(@RequestBody TaskRequest request) {
    return ResponseEntity.status(HttpStatus.CREATED)
        .body(taskService.create(request.getTitle(), request.getDescription()));
}

@Operation(summary = "Update a task")
@PutMapping("/{id}")
public TaskResponse updateTask(
        @PathVariable Long id, @RequestBody TaskRequest request) {
    return taskService.update(id, request.getTitle(), request.getDescription());
}

@Operation(summary = "Delete a task")
@DeleteMapping("/{id}")
public void deleteTask(@PathVariable Long id) {
    taskService.delete(id);
}
```

---

# Step 3 — Annotate completeTask

`completeTask` marks a task as done. A good example of a PATCH endpoint with no request body:

```java
@Operation(summary = "Mark a task as complete",
           description = "Sets completed=true on the task. Cannot be undone via this endpoint.")
@ApiResponses({
    @ApiResponse(responseCode = "200", description = "Task marked as complete",
        content = @Content(schema = @Schema(implementation = TaskResponse.class))),
    @ApiResponse(responseCode = "404", description = "Task not found",
        content = @Content(schema = @Schema(implementation = ErrorResponse.class)))
})
@PatchMapping("/{id}/complete")
public TaskResponse completeTask(
        @Parameter(description = "Task ID", example = "1")
        @PathVariable Long id) {
    return taskService.complete(id);
}
```

---

# Step 3 — Annotate filterTasks

`filterTasks` uses query params for filtering and pagination — a good place to document multiple `@Parameter`s:

```java
@Operation(summary = "Filter tasks with pagination",
           description = "Filter tasks by completion status, title, project, or date. Supports pagination.")
@GetMapping("/filter")
public PaginationResponse<TaskResponse> filterTasks(
        @Parameter(description = "Filter by completion status") FilterTaskDto filter,
        @Parameter(description = "Pagination settings") Pagination pagination) {
    return taskService.filterTask(filter, pagination);
}
```

---
zoom: 0.85
---

# Step 4 — Annotate TaskRequest

Open `TaskRequest.java` and add `@Schema` to the class and each field:

```java
@Schema(name = "TaskRequest", description = "Data to create or update a task")
@Data
@NoArgsConstructor
@AllArgsConstructor
public class TaskRequest {

    @Schema(description = "Task title (max 100 characters)", example = "Implement login page")
    private String title;

    @Schema(description = "Detailed task description",
            example = "Create login page with email and password fields")
    private String description;
}
```

---

# Step 5 — Verify in Swagger UI

Open `http://localhost:8888/swagger-ui.html` and check:

<v-clicks>

- [ ] "Tasks" tag appears with your description
- [ ] All 7 endpoints listed with your summaries
- [ ] Expand GET `/api/tasks` — shows summary, description, and optional `completed` parameter
- [ ] Expand POST `/api/tasks` — shows request body schema with `title` and `description` fields
- [ ] Expand GET `/api/tasks/{id}` — shows 200 and 404 responses with schemas
- [ ] Expand PATCH `/api/tasks/{id}/complete` — shows `id` path parameter
- [ ] Click "Try it out" on GET `/api/tasks` — execute and see results

</v-clicks>

---

# Troubleshooting

| Issue | Likely Cause | Fix |
|-------|-------------|-----|
| 404 on `/swagger-ui.html` | Dependency not downloaded | Check pom.xml, re-run Maven |
| Wrong `@ApiResponse` import | Imported from wrong package | Use `io.swagger.v3.oas.annotations.responses.ApiResponse` |
| Fields show no descriptions | `@Schema` on wrong DTO | Check you annotated the right class |
| `@Tag` not showing | Forgot to add it | Add `@Tag(name = "Tasks")` to controller class |
| Compilation error on `@RequestBody` | Name clash | Use full `@io.swagger.v3.oas.annotations.parameters.RequestBody` |

---

# ✓ Lab Complete

**Open `/swagger-ui.html`. Your API is now self-documenting.**

<v-click>

Every time you add or change an endpoint, the documentation updates automatically — no manual docs to maintain.

The next time a teammate asks "what does this API do?", send them the Swagger URL.

</v-click>

<!--
10-minute break after this lab before moving to Liquibase.
-->
