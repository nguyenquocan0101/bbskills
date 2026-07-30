---
name: bbskills
description: "Portable collection of reusable agent workflows for brainstorming, specification, planning, implementation, debugging, CI/CD, frontend design, problem solving, and skill creation. Use when a task matches one of the bundled workflows or when the user asks to choose the right workflow."
---

# bbskills — Basic Agent Skill Collection

Use this skill as a lightweight dispatcher for the workflows bundled under `skills/`.
Load only the component skill needed for the current task; do not load the entire collection
unless the user explicitly asks for a catalog or comparison.

## Component catalog

| Component | Use it for |
| --- | --- |
| `brainstorm` | Explore alternatives, clarify a design, and write a brainstorm report plus spec |
| `spec` | Write a specification directly when the direction is already decided |
| `plan` | Research, split work into phases, review the plan, and hand off implementation |
| `cook` | Implement a plan phase by phase with testing and review |
| `fix` | Scout, diagnose, fix, review, and finalize a bug fix |
| `cicd` | Scaffold or audit Docker, registry, and deployment pipelines |
| `frontend-mindset` | Apply practical frontend engineering and product implementation judgment |
| `design-taste-frontend` | Design polished, distinctive frontend interfaces |
| `minimalist-ui` | Build restrained, focused, minimalist user interfaces |
| `problem-solving` | Route a stuck problem to an appropriate reasoning technique |
| `skill-creator` | Create, update, validate, and improve reusable skills |

## Routing procedure

1. Identify the smallest matching component from the catalog.
2. Read that component's `skills/{component}/SKILL.md` completely before acting.
3. Read only the direct references, scripts, or assets required by that component.
4. Follow the component workflow and preserve its artifact and verification requirements.
5. If a referenced agent, hook, MCP tool, TodoWrite action, or slash command is unavailable,
   perform the closest equivalent inline and state the fallback briefly.

When a task spans multiple components, use the smallest useful sequence. For a novel feature,
prefer `brainstorm` → `spec` or `plan` → `cook`; for an already decided feature, use `spec` or
`plan` directly. Use `problem-solving` when the work is blocked or the failure mode is unclear.

## Portability

The bundled workflows are Markdown-first and can be used by Codex, another agent host, or a
custom Node application. Host-specific commands are integration points, not prerequisites for
the underlying reasoning. Replace unavailable host actions with equivalent local actions while
keeping the workflow's safety gates, artifacts, tests, and review steps intact.
