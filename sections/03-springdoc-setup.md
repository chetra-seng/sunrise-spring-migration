---
layout: center
---
# SpringDoc OpenAPI Setup

## Adding Swagger to Your Spring Boot Project

---

# The Dependency

Add one dependency to `pom.xml` — that's it:

```xml
<dependencies>
    <!-- existing dependencies ... -->

    <!-- Swagger / OpenAPI documentation -->
    <dependency>
        <groupId>org.springdoc</groupId>
        <artifactId>springdoc-openapi-starter-webmvc-ui</artifactId>
        <version>3.0.2</version>
    </dependency>
</dependencies>
```

<v-click>

After adding this and restarting, open your browser:

- `http://localhost:8888/swagger-ui.html` → Swagger UI
- `http://localhost:8888/v3/api-docs` → Raw OpenAPI JSON

</v-click>

<!--
No other configuration required for basic setup. Spring Boot auto-detects the library and generates the spec by scanning your controllers.
-->

---

# Auto-Detection in Action

SpringDoc scans your controllers at startup and generates documentation automatically.

Before you add a single annotation, it already knows:

<v-clicks>

- Every `@RestController` class
- Every `@GetMapping`, `@PostMapping`, etc. method
- The request/response types from method signatures
- Path variables and request parameters from `@PathVariable`, `@RequestParam`

</v-clicks>

<v-click>

**Try it now:** Add the dependency, start the app, open Swagger UI. Your endpoints are already there — they just don't have descriptions yet.

</v-click>

---

# Customizing the OpenAPI Info

Add an `OpenApiConfig` bean to customize the title, description, and version shown in Swagger UI:

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
                    .name("Sunrise Team")
                    .email("team@sunrise.dev")));
    }
}
```

<!--
This is optional but professional. The title and description appear at the top of Swagger UI.
-->

---

# Spring Security + Swagger UI

The Task Flow API has Spring Security — visiting `/swagger-ui.html` returns **401** by default. Two fixes needed:

**1 — Permit Swagger paths in `SecurityConfig.java`** (before `.anyRequest().authenticated()`):

```java
.requestMatchers("/swagger-ui.html", "/swagger-ui/**", "/v3/api-docs/**").permitAll()
```

<v-click>

**2 — Add a JWT security scheme to `OpenApiConfig.java`** (enables the "Authorize" button):

```java
// Additional imports: io.swagger.v3.oas.models.Components,
//   io.swagger.v3.oas.models.security.{SecurityRequirement, SecurityScheme}

.addSecurityItem(new SecurityRequirement().addList("Bearer Authentication"))
.components(new Components()
    .addSecuritySchemes("Bearer Authentication",
        new SecurityScheme()
            .type(SecurityScheme.Type.HTTP)
            .scheme("bearer")
            .bearerFormat("JWT")))
```

</v-click>

<v-click>

You can now open Swagger UI without a token, click **Authorize**, paste your JWT, and call secured endpoints directly from the browser.

</v-click>

---

# application.properties Configuration

Common Swagger settings you might need:

```properties
# Change the Swagger UI path (default: /swagger-ui.html)
springdoc.swagger-ui.path=/docs

# Change the OpenAPI spec path (default: /v3/api-docs)
springdoc.api-docs.path=/api-docs

# Disable Swagger in production (useful for security)
springdoc.swagger-ui.enabled=false
springdoc.api-docs.enabled=false

# Sort endpoints alphabetically
springdoc.swagger-ui.operationsSorter=alpha

# Group endpoints by tag
springdoc.swagger-ui.tagsSorter=alpha
```

<v-click>

For now, defaults are fine. In production, consider disabling Swagger UI and only exposing the raw spec to internal tools.

</v-click>

---
zoom: 0.80
---

# What the Raw OpenAPI Spec Looks Like

`GET /v3/api-docs` returns JSON that looks like this:

```json
{
  "openapi": "3.0.1",
  "info": {
    "title": "Task Flow API",
    "version": "v1.0"
  },
  "paths": {
    "/api/tasks": {
      "get": {
        "tags": ["task-controller"],
        "operationId": "findAll",
        "responses": {
          "200": {
            "description": "OK",
            "content": {
              "application/json": {
                "schema": {
                  "type": "array",
                  "items": { "$ref": "#/components/schemas/TaskResponse" }
                }
              }
            }
          }
        }
      }
    }
  }
}
```

This is the standard format that any OpenAPI-compatible tool can read.

---

# Swagger UI — A Quick Tour

When you open `/swagger-ui.html` you'll see:

<v-clicks>

**Top section** — API title, description, version from your `OpenAPI` bean

**Tags** — group name for each controller (default: class name, lowercase)

**Endpoints** — each endpoint shows: HTTP method badge, path, and summary

**Expand an endpoint** — shows: parameters, request body schema, response schemas

**"Try it out"** — fill in values and send a real HTTP request from the browser

**"Authorize"** — enter tokens for secured APIs (relevant when you add Spring Security)

</v-clicks>

<!--
Live demo here is very effective. Walk through what each part does before annotating anything.
-->
