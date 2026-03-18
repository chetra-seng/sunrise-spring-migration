---
theme: seriph
background: https://images.unsplash.com/photo-1555066931-4365d14bab8c?w=1920&h=1080&fit=crop
title: Documentation, Migration & Review
class: text-center
transition: slide-left
mdc: true
favicon: https://spring.io/favicon.svg
controls: true
download: true
info: |
  A complete guide to API documentation with Swagger/OpenAPI and database
  migration with Liquibase — making your Spring Boot Task Flow API production-ready.

keywords: Swagger, OpenAPI, Liquibase, Spring Boot, Java, Database Migration, API Documentation
author: Sunrise Java Course
description: Sunrise Java Course — API documentation with Swagger/OpenAPI and database schema management with Liquibase for the Task Flow API.
themeConfig:
  primary: '#6db33f'

seoMeta:
  ogType: website
  ogTitle: Documentation, Migration & Review
  ogDescription: API documentation with Swagger/OpenAPI and database migration with Liquibase — Sunrise Java Course.
  ogImage: https://images.unsplash.com/photo-1555066931-4365d14bab8c?w=1200&h=630&fit=crop

htmlAttrs:
  lang: en
  dir: ltr
---

# Documentation, Migration & Review

Sunrise Java Course

<div class="pt-12">
  <span class="px-2 py-1">
    From working API to production-ready — with Swagger docs and Liquibase migrations
  </span>
</div>

---

# What We'll Learn

- 📄 **Swagger / OpenAPI** — auto-generate beautiful API docs from your code
- 🏷️ **Annotations** — `@Tag`, `@Operation`, `@Schema` and more
- 🗂️ **Liquibase** — manage database schema changes safely across environments
- 🔄 **Changelogs** — write, run, and roll back migrations in YAML/XML
- 🚀 **Best practices** — naming, ordering, environments, and CI/CD integration

---
src: ./sections/01-intro-module-overview.md
---

---
src: ./sections/02-why-api-documentation.md
---

---
src: ./sections/03-springdoc-setup.md
---

---
src: ./sections/04-swagger-annotations.md
---

---
src: ./sections/05-documenting-models.md
---

---
src: ./sections/06-swagger-lab.md
---

---
src: ./sections/07-why-database-migrations.md
---

---
src: ./sections/08-liquibase-intro.md
---

---
src: ./sections/09-liquibase-setup.md
---

---
src: ./sections/10-changelog-fundamentals.md
---

---
src: ./sections/11-liquibase-lab-1.md
---

---
src: ./sections/12-advanced-changelogs.md
---

---
src: ./sections/13-contexts-labels.md
---

---
src: ./sections/14-liquibase-lab-2.md
---

---
src: ./sections/15-rollback-strategies.md
---

---
src: ./sections/16-best-practices-environments.md
---

---
src: ./sections/17-liquibase-lab-3.md
---

---
src: ./sections/18-course-recap.md
---

---
layout: center
class: text-center
---

# Course Complete!

You've made your Task Flow API production-ready with documentation and safe schema management.

<div class="text-left pt-8">

**What you added:**
- `springdoc-openapi` dependency → Swagger UI at `/swagger-ui.html`
- `@Tag`, `@Operation`, `@ApiResponse`, `@Schema` → rich API documentation
- `liquibase-core` dependency → schema version control
- `db/changelog/` directory → versioned SQL migrations in YAML/XML
- `ddl-auto=validate` or `none` → no more surprise schema changes

**What stayed the same:** `TaskController`, `TaskServiceImpl`, `TaskMapper` — untouched

**Resources:**
- SpringDoc OpenAPI: <a href="https://springdoc.org/" target="_blank">springdoc.org</a>
- Liquibase Docs: <a href="https://docs.liquibase.com/" target="_blank">docs.liquibase.com</a>
- OpenAPI Specification: <a href="https://swagger.io/specification/" target="_blank">swagger.io/specification</a>

</div>
