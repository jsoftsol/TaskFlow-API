# Product Requirements Document

> Reverse-engineered from the current codebase (as of 2026-08-09). This describes what is actually built, not a pre-existing spec. Sections marked **[assumption]** are inferred from code shape rather than stated anywhere; correct them as the real product intent becomes clear.

## Overview

A multi-tenant SaaS backend for project/task management. Each customer organization ("Company") has its own users, projects, and tasks, isolated from other companies. Access is API-only (Sanctum token auth); no bundled frontend.

## Target users **[assumption]**

Small-to-mid teams inside a company who need to track projects and assign tasks to members, with role-based permissions distinguishing who can manage work vs. who can only act on what's assigned to them.

## Core entities

- **Company**: the tenant. Owns users and projects.
- **User**: belongs to exactly one company. Authenticates via Sanctum tokens. Has a role (Admin / Manager / Developer) via Spatie permissions.
- **Project**: belongs to a company. Owns tasks.
- **Task**: belongs to a project (and transitively a company), optionally assigned to a user. Has a status (`pending` / `in_progress` / `completed`).

## Roles & permissions (as implemented)

| Role | Task visibility | Task actions |
|---|---|---|
| Admin | all tasks in own company | view, update, delete, assign |
| Manager | all tasks in own company | view, update, delete, assign |
| Developer | only tasks assigned to self | view, update own |

Admin/Manager are currently equivalent in permission logic, with no distinction enforced anywhere yet **[gap]**.

## Features implemented

- Registration and login issuing Sanctum tokens (`POST /api/v1/register`, `/login`), logout (`POST /api/v1/logout`).
- Project listing/creation scoped to the caller's company (`GET|POST /api/v1/projects`).
- Task creation within a company's project, task view/update gated by `TaskPolicy` (`/api/v1/tasks`).
- Role-gated web route example (`/admin`, requires `admin` role).

## Features declared but not working

(none currently: as of 2026-08-09 the previously-listed gaps, the `assign` route, project update/destroy, task index/destroy, and mass-assignment bugs on `Project`/`Task`, are all fixed. See `MEMORIES.md` for detail.)

## Explicit non-goals (current state, not necessarily permanent)

- No application frontend UI (Blade/Livewire/Inertia/SPA): Tailwind/Vite are wired up but unused. The `/` route (`resources/views/welcome.blade.php`) is a static, self-contained informational landing page (project name/tagline/architecture highlights), not an app UI; it doesn't use Tailwind/Vite or talk to the API.
- No cross-company sharing of projects/tasks: a task and its project always belong to one company.
- No billing/subscription logic yet.
- No email verification, password reset, or 2FA flows yet.

## Open product questions **[assumption, needs real answers]**

- Should Admin and Manager have distinct permissions, or are they meant to be equivalent?
- Should a user ever belong to more than one company?
- What should happen to tasks/projects when a company or user is deleted (cascade behavior beyond what migrations define)?
- Is an admin UI or customer-facing UI planned, or does this stay API-only?
