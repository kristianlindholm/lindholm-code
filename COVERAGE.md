# Coverage — language support matrix

This file records what each supported language ships, and marks every gap as
intentional. It is the source of truth for the Definition of Done's coverage-parity
clause in `CLAUDE.md`: gaps here are deliberate, not accidental.

## The standard set

Full support for a language means all five artifacts exist and are wired into
`new-project`'s tech-choice table:

1. **Rules** — `Rules/Stack_specific/<lang>/` (coding-style, testing, security, ...).
2. **Patterns skill** — language/framework idioms and structure.
3. **Testing skill** — language-specific test patterns on top of Global `testing.md`.
4. **Reviewer agent** — enforces standards on a diff.
5. **Build-resolver agent** — fixes build/compile/dependency failures.

Not every language needs all five. The set below is the target; the matrix records
where a language deliberately stops short.

## Gap policy — build on demand, not on spec

A missing artifact is filled when a real project uses the language **and** the generic
coverage (Global rules + the reviewer + the main-loop build workflow) proves
insufficient — never speculatively. Rationale:

- The **reviewer** is the highest-leverage per-language artifact and already exists for
  almost every language. It is the priority when filling a gap.
- A **patterns or testing skill** earns its place only when it carries non-obvious,
  recurring, language-specific guidance (the `create-skill` justification test). A thin
  skill that restates the Global rules is bloat, not coverage.
- A dedicated **build-resolver** pays off most for complex compilers (Rust, C++,
  Java/Gradle), which already have one. Interpreted/dynamic stacks fall back to the
  generic flow, which handles their import/dependency/syntax failures well.

## Matrix

Cells name the provisioning artifact, or `—` for an intentional gap. "shared" marks
coverage provided by a cross-stack artifact rather than a language-specific one.

| Language | Rules | Patterns | Testing | Reviewer | Build-resolver | Tier |
|---|---|---|---|---|---|---|
| React | yes | react-patterns | react-testing | react-reviewer | react-build-resolver | Full |
| Django | yes | django-patterns | python-testing | django-reviewer | django-build-resolver | Full |
| Go | yes | golang-patterns | golang-testing | go-reviewer | go-build-resolver | Full |
| Rust | yes | rust-patterns | rust-testing | rust-reviewer | rust-build-resolver | Full |
| Kotlin | yes | kotlin-patterns | kotlin-testing | kotlin-reviewer | kotlin-build-resolver | Full |
| Swift | yes | swiftui-patterns | swift-protocol-di-testing | swift-reviewer | swift-build-resolver | Full |
| Python | yes | python-patterns | python-testing | python-reviewer | — | Partial |
| FastAPI | yes | fastapi-patterns | python-testing | fastapi-reviewer | — | Partial |
| TypeScript | yes | frontend-patterns, backend-patterns | shared (e2e-testing, react-testing) | typescript-reviewer | typescript-build-resolver | Partial |
| C++ | yes | — | cpp-testing | cpp-reviewer | cpp-build-resolver | Partial |
| C# | yes | — | csharp-testing | csharp-reviewer | — | Partial |
| F# | yes | — | fsharp-testing | fsharp-reviewer | — | Partial |
| Dart / Flutter | yes | dart-flutter-patterns | — | flutter-reviewer | dart-build-resolver | Partial |
| Java | yes | — | — | java-reviewer | java-build-resolver | Partial |
| PHP | yes | — | — | php-reviewer | — | Partial |
| Perl | yes | perl-patterns | perl-testing | — | — | Partial |
| Angular | yes | angular-developer | — | shared (typescript-reviewer) | shared (typescript-build-resolver) | Partial |
| Vue | yes | shared (frontend-patterns, vite-patterns) | — | shared (typescript-reviewer) | shared (typescript-build-resolver) | Partial |
| Nuxt | yes | nuxt4-patterns | — | shared (typescript-reviewer) | shared (typescript-build-resolver) | Partial |
| Ruby | yes | — | — | — | — | Minimal |
| HarmonyOS / ArkTS | yes | — | — | — | — | Minimal |
| General web | yes | frontend-patterns, vite-patterns | e2e-testing | — | shared (typescript-build-resolver) | Support |

## Notes on intentional gaps

- **No build-resolver for Python / FastAPI / C# / F# / PHP.** Their build failures are
  import, dependency, or syntax issues the generic flow resolves; a dedicated specialist
  adds little. Django is the exception (migrations, settings) and has one.
- **No dedicated TypeScript testing skill.** `e2e-testing` plus `react-testing` cover the
  TypeScript testing surface; a standalone `typescript-testing` skill would duplicate them.
- **TS-framework reviewers are shared.** Angular, Vue, and Nuxt are TypeScript stacks and
  use `typescript-reviewer`; no framework-specific reviewer exists by design until a
  project shows the generic TypeScript review is insufficient.
- **No patterns/testing skills for Java, C#, C++, F#, PHP.** The Global rules plus each
  language's reviewer carry the standard. A language patterns/testing skill is added only
  when it would hold genuinely language-specific depth.
- **Ruby and ArkTS are rules-only by design** (minimal support). They remain selectable in
  `new-project`; skills and agents are added on demand, not pre-built.
- **Shared web design rules.** `Rules/Stack_specific/design/` (`design-quality.md`,
  `design-principles.md`) is a framework-agnostic ruleset, not a language row. `new-project`
  provisions it alongside every web frontend stack (React, Angular, Vue, Nuxt, General web),
  so design standards apply regardless of framework. Native-UI stacks (Swift, Kotlin, Flutter,
  ArkTS) do not receive it — these rules are web-specific (CSS tokens, web WCAG, viewport).
- **TypeScript backend sub-stack skills.** Beyond the general `frontend-patterns` /
  `backend-patterns` in the matrix, the TypeScript backend surface also ships targeted skills
  provisioned when the stack calls for them: `prisma-patterns` (Prisma ORM, incl. Prisma 7)
  and `nestjs-patterns` (NestJS architecture). These are not per-language rows; they attach
  to a project by its chosen libraries.
- **Infrastructure skills (stack-agnostic, library-attached).** A named category inside
  `Skills/Stack_specific/`: `docker-patterns`, `deployment-patterns`, `database-migrations`,
  `e2e-testing`, `postgres-patterns`, and `redis-patterns` are not tied to a programming
  language. They live in `Stack_specific/` because they are provisioned per-project — a
  project gets them when its stack includes the corresponding tool or infrastructure choice,
  regardless of which language it is written in. They have no matrix row; `new-project`
  attaches them by tooling choice, not by language.
