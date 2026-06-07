---
name: documentation
description: Document the why, not the what. Covers inline comments, API docs, READMEs, changelogs, and agent-context files (CLAUDE.md). Use when adding or changing documentation, when onboarding teammates or agents, when explaining the same thing repeatedly, or when shipping a feature that changes behaviour. For ADRs, defer to `grill-with-docs/ADR-FORMAT.md`.
---

# Documentation

**Document the *why*, not the *what*.** Code shows what was built. Documentation explains why it was built this way and what alternatives were rejected. That context is what future humans and agents actually need.

For architectural decisions specifically, this skill defers to the existing convention in [`../grill-with-docs/ADR-FORMAT.md`](../grill-with-docs/ADR-FORMAT.md). Don't write a competing ADR template here.

## When NOT to document

- Don't document obvious code. Comments restating what the code already says are noise.
- Don't write docs for throwaway prototypes.
- Don't leave TODO comments for things you should just do now.
- Don't leave commented-out code "in case we need it". Git has the history.

## ADRs (pointer)

ADRs capture decisions that are hard to reverse, surprising without context, and the result of a real trade-off. Format, location, and full criteria live in [`../grill-with-docs/ADR-FORMAT.md`](../grill-with-docs/ADR-FORMAT.md). The short version:

- Live in `docs/adr/`, sequentially numbered (`0001-slug.md`).
- A single paragraph is enough. Sections are optional.
- Don't delete superseded ADRs; write a new one that references and supersedes them.

## Inline comments

### When to comment

Comment the *why*, not the *what*:

```ts
// BAD: Restates the code
// Increment counter by 1
counter += 1;

// GOOD: Explains non-obvious intent
// Sliding window: reset counter at window boundary, not on a fixed
// schedule, to prevent burst attacks at window edges
if (now - windowStart > WINDOW_SIZE_MS) {
  counter = 0;
  windowStart = now;
}
```

### When NOT to comment

```ts
// Don't restate self-explanatory code
function calculateTotal(items: CartItem[]): number {
  return items.reduce((sum, item) => sum + item.price * item.quantity, 0);
}

// Don't leave TODOs for things you should just do now
// TODO: add error handling   ← Just add it.

// Don't leave commented-out code
// const oldImpl = () => { ... }   ← Delete it. git has the history.
```

### Document known gotchas

Hidden constraints, subtle invariants, or workarounds for specific bugs are exactly what comments are for:

```ts
/**
 * IMPORTANT: This must be called before the first render.
 * If called after hydration, it causes a flash of unstyled content
 * because the theme context isn't available during SSR.
 *
 * See ADR-0003 for the full design rationale.
 */
export function initializeTheme(theme: Theme): void {
  // ...
}
```

## API documentation

For public APIs (REST, GraphQL, library interfaces), the docs are part of the contract.

### Inline doc comments (preferred for typed languages)

```ts
/**
 * Creates a new task.
 *
 * @param input - Task creation data (title required, description optional).
 * @returns The created task with server-generated ID and timestamps.
 * @throws {ValidationError} If title is empty or exceeds 200 characters.
 * @throws {AuthenticationError} If the user is not authenticated.
 *
 * @example
 * const task = await createTask({ title: 'Buy groceries' });
 * console.log(task.id); // "task_abc123"
 */
export async function createTask(input: CreateTaskInput): Promise<Task> {
  // ...
}
```

### OpenAPI / Swagger for REST

```yaml
paths:
  /api/tasks:
    post:
      summary: Create a task
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/CreateTaskInput'
      responses:
        '201':
          description: Task created
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/Task'
        '422':
          description: Validation error
```

## README structure

Every project should have a README that covers:

```markdown
# Project Name

One-paragraph description of what this project does and who it's for.

## Quick start
1. Clone the repo
2. Install dependencies: `npm install`
3. Set up environment: `cp .env.example .env`
4. Run the dev server: `npm run dev`

## Commands
| Command | Description |
|---------|-------------|
| `npm run dev`   | Start development server |
| `npm test`      | Run tests |
| `npm run build` | Production build |
| `npm run lint`  | Run linter |

## Architecture
Brief overview of the project structure and key design decisions.
Link to ADRs in `docs/adr/` for detail.

## Contributing
How to contribute, coding standards, PR process.
```

If the README is the first thing a new contributor reads, it should answer "what is this and how do I run it" within thirty seconds.

## Changelog

For shipped features, keep a `CHANGELOG.md` in [Keep a Changelog](https://keepachangelog.com) format:

```markdown
# Changelog

## [1.2.0] - 2026-01-20
### Added
- Task sharing: users can share tasks with team members (#123)
- Email notifications for task assignments (#124)

### Fixed
- Duplicate tasks appearing when rapidly clicking create (#125)

### Changed
- Task list now loads 50 items per page (was 20) for better UX (#126)
```

## Documentation for agents

Agents read documentation differently from humans. They have no institutional memory, so written context is the *only* context. Treat these as load-bearing:

- **`CLAUDE.md` / rules files**: project conventions, terminology, hard "don'ts". This is the highest-leverage file you can write because it loads on every session.
- **`CONTEXT.md`**: domain glossary, the project's vocabulary for talking about itself. Maintained inline by `grill-with-docs` and `improve-codebase-architecture`.
- **ADRs in `docs/adr/`**: frozen decisions. Stop agents (and humans) re-deciding the same thing every six months.
- **Inline gotchas**: the `IMPORTANT:` comments above. Prevent agents from falling into known traps.

## Honesty

- **"The code is self-documenting" is half-true.** Code shows *what*. It doesn't show *why*, what alternatives were rejected, or what constraints apply. Document the half it can't show.
- **"We'll write docs when the API stabilises" is backwards.** APIs stabilise faster when you document them. The doc is the first test of the design.
- **"Nobody reads docs" is false.** Agents do. Future engineers do. Your three-months-later self does.
- **"Comments get outdated" only applies to *what* comments.** Comments on *why* are stable. Comments on *what* drift, which is why you only write the former.
- **A 10-minute ADR prevents a 2-hour debate** about the same decision six months later.
