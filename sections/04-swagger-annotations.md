---
layout: center
---
# Swagger Annotations

## Making Your Docs Meaningful

---

# The Four Core Annotations

SpringDoc reads these annotations to generate your documentation:

<v-clicks>

| Annotation | Put it on | What it does |
|-----------|----------|-------------|
| `@Tag` | Controller class | Groups endpoints under a name in Swagger UI |
| `@Operation` | Controller method | Describes what the endpoint does |
| `@Parameter` | Method parameter | Describes a path variable or query param |
| `@ApiResponse` | Controller method | Describes a possible HTTP response |

</v-clicks>

<v-click>

Think of them as labels you write **once** — Swagger reads them and builds the UI for you.

</v-click>

---

# @Tag — Grouping Endpoints

`@Tag` goes on the controller class. It gives your endpoints a readable group name.

````md magic-move
```java
// Before: SpringDoc generates an ugly default name
@RestController
@RequestMapping("/api/tasks")
@RequiredArgsConstructor
public class TaskController {
    // endpoints grouped as "task-controller"
}
```

```java
// After: clean group name with description
@Tag(name = "Tasks", description = "Endpoints for managing tasks")
@RestController
@RequestMapping("/api/tasks")
@RequiredArgsConstructor
public class TaskController {
    // endpoints grouped as "Tasks" with description
}
```
````

<!--
The "before" state is what auto-detection gives you. The "after" is what a single annotation adds.
-->

---

# @Operation — Describing an Endpoint

`@Operation` goes on each controller method. It provides the summary (one line) and description (paragraph).

```java
@Operation(
    summary = "Get all tasks",
    description = "Returns a list of all tasks. " +
                  "Use /filter for paginated results with title, project, date, and completion filters."
)
@GetMapping
public List<TaskResponse> getAllTask(
        @RequestParam(required = false) Boolean completed) {
    return taskService.findAll();
}
```

<v-click>

`summary` → the short line shown in the collapsed endpoint list

`description` → the full paragraph shown when the endpoint is expanded

</v-click>

---

# @Parameter — Documenting Path Variables & Query Params

`@Parameter` annotates `@RequestParam` and `@PathVariable` to add descriptions and examples.

```java
@Operation(summary = "Mark a task as complete")
@PatchMapping("/{id}/complete")
public TaskResponse completeTask(
    @Parameter(
        description = "ID of the task to complete",
        example = "1"
    )
    @PathVariable Long id) {

    return taskService.complete(id);
}
```

<!--
The example values appear in Swagger UI's "Try it out" form with pre-filled values.
-->

---
zoom: 0.85
---

# @ApiResponse — Documenting Responses

`@ApiResponse` describes each HTTP response code your endpoint can return:

```java
@Operation(summary = "Get task by ID")
@ApiResponses({
    @ApiResponse(
        responseCode = "200",
        description = "Task found",
        content = @Content(schema = @Schema(implementation = TaskResponse.class))
    ),
    @ApiResponse(
        responseCode = "404",
        description = "Task not found — no task exists with this ID",
        content = @Content(schema = @Schema(implementation = ErrorResponse.class))
    )
})
@GetMapping("/{id}")
public TaskResponse getTaskById(@PathVariable Long id) {
    return taskService.findById(id);
}
```

<v-click>

This tells API consumers exactly what to expect — they don't have to guess what a 404 body looks like.

</v-click>

---
zoom: 0.85
---

# Putting It Together — Full Annotated Method

```java
@Tag(name = "Tasks", description = "Endpoints for managing tasks")
@RestController
@RequestMapping("/api/tasks")
@RequiredArgsConstructor
public class TaskController {

    @Operation(
        summary = "Create a new task",
        description = "Creates a task and saves it to the database. Returns the saved task with its generated ID."
    )
    @ApiResponses({
        @ApiResponse(responseCode = "201", description = "Task created successfully",
            content = @Content(schema = @Schema(implementation = TaskResponse.class))),
        @ApiResponse(responseCode = "400", description = "Invalid request body — check required fields",
            content = @Content(schema = @Schema(implementation = ErrorResponse.class)))
    })
    @PostMapping
    public ResponseEntity<TaskResponse> createTask(@RequestBody TaskRequest request) {
        return ResponseEntity.status(HttpStatus.CREATED)
            .body(taskService.create(request.getTitle(), request.getDescription()));
    }
}
```

---
zoom: 0.85
---

# @RequestBody Annotation for Swagger

When your endpoint accepts a body, document the `@RequestBody` with `@io.swagger.v3.oas.annotations.parameters.RequestBody`:

```java
@Operation(summary = "Update a task")
@PutMapping("/{id}")
public TaskResponse updateTask(
        @PathVariable Long id,
        @io.swagger.v3.oas.annotations.parameters.RequestBody(
            description = "Updated task data — title and description.",
            required = true,
            content = @Content(schema = @Schema(implementation = TaskRequest.class))
        )
        @RequestBody TaskRequest request) {
    return taskService.update(id, request.getTitle(), request.getDescription());
}
```

<v-click>

Note: The `@RequestBody` from Spring and the one from Swagger have the same name. Use the full package path for the Swagger one to avoid confusion.

</v-click>

---

# Annotation Import Reference

Quick cheat sheet — all from `io.swagger.v3.oas.annotations`:

```java
import io.swagger.v3.oas.annotations.Operation;
import io.swagger.v3.oas.annotations.Parameter;
import io.swagger.v3.oas.annotations.media.Content;
import io.swagger.v3.oas.annotations.media.Schema;
import io.swagger.v3.oas.annotations.responses.ApiResponse;
import io.swagger.v3.oas.annotations.responses.ApiResponses;
import io.swagger.v3.oas.annotations.tags.Tag;
```

<v-click>

Your IDE will suggest the correct import. If you see two `@ApiResponse` options, pick the one from `io.swagger.v3`.

</v-click>

<!--
Common beginner mistake: importing the wrong @ApiResponse from spring-web. Mention this preemptively.
-->
