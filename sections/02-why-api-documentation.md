---
layout: center
---
# Why API Documentation?

## The Problem with Undocumented APIs

---

# Imagine You're a Frontend Developer

Your backend teammate hands you a new Task Flow API. They say: "It's ready, go use it."

<v-clicks>

You ask: **"What endpoints exist?"** → "Check the controller."

You ask: **"What does the request body look like?"** → "Check the DTO."

You ask: **"What does it return on error?"** → "I don't know, just try it."

You ask: **"What's the base URL?"** → "It depends on environment."

</v-clicks>

<v-click>

This is the reality of undocumented APIs. It wastes everyone's time.

</v-click>

<!--
Ask the class: "Has anyone been in this situation — either side?" Good discussion prompt.
-->

---

# With Documentation

The frontend developer opens a browser and sees this:

<div class="mt-4 border-2 border-green-400 rounded-lg p-4 bg-green-50 dark:bg-green-950">

**Swagger UI** — auto-generated from your annotations:

- Every endpoint listed with HTTP method, path, and description
- Request body schema with field types, required/optional, example values
- Response schemas for success and error cases
- A "Try it out" button to make live requests
- No setup required — it reads your code

</div>

<v-click>

This is what `springdoc-openapi` gives you — for free, from annotations you write once.

</v-click>

---

# OpenAPI vs Swagger — What's the Difference?

You'll often hear both terms used interchangeably. Here's the relationship:

<div class="grid grid-cols-2 gap-6 mt-4">

<div class="border rounded-lg p-4 border-blue-300 bg-blue-50 dark:bg-blue-950">

**OpenAPI Specification**
- The standard (version 3.x)
- A JSON/YAML format for describing REST APIs
- Vendor-neutral — any tool can read it
- Formerly called "Swagger Specification"

</div>

<div class="border rounded-lg p-4 border-green-300 bg-green-50 dark:bg-green-950">

**Swagger**
- A set of tools built around OpenAPI
- Swagger UI — the browser interface
- Swagger Editor — write specs visually
- Made by SmartBear, donated the spec to OpenAPI Initiative

</div>

</div>

<v-click>

**In practice:** OpenAPI is the spec, Swagger UI is what you see in the browser. When people say "add Swagger", they mean "generate an OpenAPI spec and display it in Swagger UI."

</v-click>

---
zoom: 0.9
---

# Two Ways to Write API Docs

<div class="grid grid-cols-2 gap-6 mt-2">

<div>

### Code-first (our approach)
Write the API in Java, annotate it, and **generate** the OpenAPI spec automatically.

```java
@Operation(summary = "Get all tasks")
@GetMapping("/api/tasks")
public ResponseEntity<List<TaskResponse>> findAll() {
    ...
}
```

✅ Always in sync with the code
✅ Less work — one source of truth
✅ Easy to start

</div>

<div>

### Design-first
Write the OpenAPI YAML spec **first**, then generate or implement the code.

```yaml
paths:
  /api/tasks:
    get:
      summary: Get all tasks
      responses:
        '200':
          description: Success
```

✅ Good for teams aligning before coding
✅ API contract before implementation
⚠️ Spec can drift from code

</div>

</div>

<!--
We use code-first. It's the most common approach for existing projects and beginners.
-->

---

# What We'll Add

By the end of the Swagger section, your API will have:

<v-clicks>

- `GET /v3/api-docs` → raw OpenAPI JSON spec
- `GET /swagger-ui.html` → interactive browser UI
- Full documentation for all 10 Task Flow endpoints
- Request/response schemas with field descriptions
- Example values for all DTOs

</v-clicks>

<v-click>

And you'll write **zero extra code** — just annotations on classes you already have.

</v-click>

<!--
This motivates keeping going — the payoff is clear and concrete.
-->
