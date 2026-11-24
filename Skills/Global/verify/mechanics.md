# Verify — mechanics

How to carry out [`SKILL.md`](SKILL.md)'s steps in this environment. Everything here expires; the
method does not. Re-check anything below before relying on it.

## Starting an application

`.claude/launch.json` at the project root is the start configuration. Its shape:

```json
{
  "version": "0.0.1",
  "configurations": [
    { "name": "<unique-name>", "runtimeExecutable": "npm", "runtimeArgs": ["run", "dev"], "port": 3000 }
  ]
}
```

`preview_start` with the configuration's `name` opens it in the Browser pane. Two cautions, both
observed:

- **`preview_start` resolves `launch.json` from the session's original working directory**, not the
  current one. Where they differ, start the process directly and open the URL with `navigate`.
- **A `next dev` child survives its parent being stopped** and keeps holding the port. Stopping the
  task is not stopping the server. Check the port, then kill what holds it.

Where a project has no `launch.json`, the start command is still discoverable — `package.json`
scripts, a Makefile, a documented command. Having none at all is the "nothing to verify" case.

## Reading the four channels

| Channel | How |
|---|---|
| What the process says | Redirect the start command's output to a file and read it. **This is the only channel that sees a failed dependency.** |
| What the runtime says | `read_console_messages`, `onlyErrors` for triage then the full list |
| What crossed a boundary | `read_network_requests`, which lists every request the page made and its status |
| What a person would see | `get_page_text` for content, `read_page` for the interactive tree, `find` to locate a control |

An offline application has fewer: a page served from `file://` with `connect-src 'none'` makes no
requests and has no server process, so two of the four are `n/a` by design, not missing.

## Framework noise

Lines that a clean run still emits. Excluded from "clean", and counted in the report. **Every
channel needs its own set** — the browser console is not the only one that chatters.

**Runtime channel (browser console)**

| Emitted by | Lines |
|---|---|
| React, development build | `Download the React DevTools for a better development experience` |
| Next.js with Turbopack | `[HMR] connected`, `[Fast Refresh] rebuilding`, `[Fast Refresh] done in Nms` |

**Process channel (server stdout and stderr)**

| Emitted by | Lines |
|---|---|
| npm | the run banner, two lines before anything else |
| Next.js with Turbopack | the version and `Local` / `Network` / `Environments` block, `Ready in Xs`, one `Compiling <route>` per route on first touch, and one `GET <path> <status> in Nms` per request |

A request log line is noise; the same line carrying a status you did not expect is not. Read the
status before excluding the line.

Observed on Next.js 16.2.7 and 16.2.10. A warning is never noise in either channel — a deprecation
notice is often the first sign of a breakage, so only the lines above are excluded.

## Probing without writing

The aim is an operation that forces the application to touch a dependency and then stops. A request
that **reads before it decides** does this — where one exists. One worked example, and one case where
it does not apply:

- **Authentication against a user store:** the attempt queries the store before comparing anything,
  so an attempt with an address that cannot exist — `@example.invalid` — reaches the database and
  writes nothing.
- **Authentication against a hashed secret in the environment:** touches no dependency at all. The
  comparison is local. This is not a variant of the case above; it is the absence of it.

**Where no application path reaches a dependency without a session,** probe it directly using the
application's own configuration and request shape, and label the result out-of-band. That proves the
dependency is alive. It proves nothing about the application's path to it, and the report must not
let the two read as the same thing.

Before any probe that does write, establish what the dependency actually is. A hosted datasource is
not a local file, and a write may land somewhere shared. Ask.

## Observed quirks, and why a control is mandatory

Each cost a near-miss on 2026-11-24. None is to be trusted on this table's authority — each is
re-testable in a minute by the control the method prescribes.

| Observed | What it would have caused |
|---|---|
| Service-worker registration fails in the Browser pane even for a three-line worker served with the correct type from a throwaway server | A false defect report against an application whose PWA support was fine. **Do not read service-worker state here.** |
| `read_page` returned "(empty page)" while `get_page_text` and `find` returned the full content at the same moment | Reporting a rendered page as blank. **Check the reported Viewport first: `0x0` means the pane has not sized the tab yet — re-read rather than conclude.** Under `filter: "interactive"` an empty result can also simply mean the page has no interactive elements. |
| A dead database returned HTTP 200 through an authentication endpoint, with console, network and page all clean | Reporting a completely non-functional application as working. This is why the process's own output is a channel. |

The shape of the control that settled the first one: serve a minimal equivalent from a throwaway
static server on another port, and attempt the same operation. Identical failure means the
environment.

## Known gaps

- The declaration-first discovery misses a dependency added without a new declared key — a new
  endpoint on a service already reached.
- A probe only covers a dependency the run actually touches; a subsystem never visited stays
  invisible.

Both are the suite's ground, not verify's.
