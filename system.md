# System design

This document describes the parts of the Brainstorm system and how they fit together. The language is defined in README.md, and the product vision and roadmap are in MyVision.md.

The system has five layers:
* **Language core**: turns text into data and data back into text. Every client and the server use it.
* **Programs and repetition**: copies plans and runs schedules.
* **Brainstorm service**: the central server that stores, imports, publishes and serves brainstorms.
* **Clients**: the web app for desktops and tablets, and the phone app.
* **AI**: helps people write documents and structures, and work on their tasks.

This version covers Phase 1, the smallest system that proves the idea. Later components are listed at the end.

## Platform
| Part | Choice |
|---|---|
| Runtime | .NET 10 (LTS) |
| Language core | A plain C# class library with no UI or database dependencies, tested with xUnit against the `samples` folder |
| Web | ASP.NET Core with a Blazor Web App, choosing the render mode per page |
| Editor | CodeMirror 6 through JS interop |
| Data | PostgreSQL with EF Core, storing each brainstorm's document text and its parsed model |
| Schedules | Ical.Net |
| Phone | An installable web app first, and .NET MAUI Blazor Hybrid only if that is not enough |

The language core runs everywhere the language is needed: on the server, in the browser through WebAssembly, in the AI connector and in a command line tool.

Pages choose how they are rendered:
* **Static server rendering** for program pages, the marketplace and sign up. They load fast, can be found by search engines and need no download, so someone following an author's link can start within 30 seconds.
* **Interactive WebAssembly**, or Auto, for the editor and the structured views, where flags and completion should appear as the user types.
* **Server rendering with light interactivity** for My tasks, so it stays fast on phones.

Patterns are run with `RegexOptions.NonBacktracking`. Patterns are written by users, and this engine always finishes in linear time, so a badly written pattern can not hang the system. It does not support lookarounds or backreferences, which the language already leaves out.

## Projects
All the code is in one solution, `Brainstorm.slnx`.

```
Brainstorm.slnx
src/
  Brainstorm.Language/        the language core
  Brainstorm.Programs/        programs and repetition
  Brainstorm.Web/             the Brainstorm service and server-rendered pages
  Brainstorm.Web.Client/      pages that run in the browser
  Brainstorm.Cli/             the command line tool
tests/
  Brainstorm.Language.Tests/
  Brainstorm.Programs.Tests/
samples/                      test documents for the language core
```

| Assembly | Components | Uses |
|---|---|---|
| Brainstorm.Language | Structure parser, structure model, document parser, brainstorm model, validator, formula engine, link resolver, document writer, upgrade check | A YAML reader only |
| Brainstorm.Programs | Copy engine, scheduler | Language, Ical.Net |
| Brainstorm.Web | Accounts, storage, sign in, program versions, import and export, My tasks, program publishing, program pages, the phone app. Later the AI tools, chat and MCP server | Language, Programs, Web.Client, EF Core with PostgreSQL |
| Brainstorm.Web.Client | Text editor, structured views, program updates, Copy for AI | Language |
| Brainstorm.Cli | Command line tool | Language |

How the code is split:
* The language core has no UI, database or network code, so it can run on the server, in the browser, in the command line tool and later in a language server. When it needs something from outside, such as the users and contacts the link resolver looks up, it asks through an interface that the service provides.
* Inside an assembly, each component is a folder and a namespace, such as `Brainstorm.Language.Structures`. A component only becomes its own assembly when something needs it without the rest.
* Programs are separate from the language core because only the server runs them, and Ical.Net does not need to be downloaded to the browser.
* Blazor requires pages that run in WebAssembly to be in their own project, which is `Brainstorm.Web.Client`. Everything else on the web is in `Brainstorm.Web`.
* The service stays inside the web project until it needs to be split, for example if the MCP server has to run on its own.
* The phone app is the My tasks pages of the web project, installed from the browser. It only gets its own project if it becomes a native app.

## Language core

### Structure parser
Reads a `.bss` Markdown document. Versions and change notes are read from the YAML front matter, and definitions only from `bss` fences. Everything else is kept as help text. Produces the raw definitions of types, extensions, choices and functions, and the list of versions.

### Structure model
Turns the raw definitions into a `StructureDefinition`, which holds:
* **Types**: name, alias, description, the type they extend, and their elements in declared order, which shorthand depends on.
* **Elements**: name, caption, description, type, marks, default, formula and rules.
* **Choices**: name, description, and values with a name, caption and description.
* **Functions**: return type, name, parameters, description and formula.

Structures never generate code: every brainstorm is held in the same generic classes, shaped by these definitions. The structure model resolves names and aliases, adds the built-in types and choices, applies extensions, and reads marks (`+`, `!`, `!!`), rules, defaults and formulas. It reports problems in the structure itself, such as unknown types, formulas that depend on themselves, functions that call themselves, and defaults that break their rules.

### Document parser
Reads the front matter of a `.bsd` document to find its structure, then reads the rest using a `StructureDefinition`. It recognises sections, notes, items, shorthand, lists, list item types, media, relative dates and links. It never fails: a value it can not understand is kept as written and flagged.

### Brainstorm model
The in-memory representation of a brainstorm that every other component works on: its structure, sections, notes and root items. Each item has an internal id, so links survive renaming, and the line it came from, so flags can point to it.

An item's elements are a dictionary keyed by element name, ignoring case. Each value keeps:
* the raw text, exactly as written;
* the parsed value, or nothing when the text could not be understood;
* the element's definition, or nothing when the type has no such element.

A value that could not be understood and an element the type does not have are the same case at two levels, and both keep their raw text. An unknown element also keeps the lines indented below it. So nothing written is ever lost, and an element renamed by a new structure version still holds its old value.

Tags are found in the raw text when they are needed, so they are not stored separately.

### Validator
Checks a brainstorm model against its structure: expected values, ranges, patterns, conditions, mistyped types and calculated values written in a document. It checks at three levels:
* **Hints**, while writing. Nothing is blocked.
* **Referential**, for imports where every link must be found.
* **Strict**, for strict imports, publishing and scheduling. Only the plan is checked when publishing and scheduling.

### Formula engine
Works out calculated values and conditions, including totals over lists (`sum`, `count`, `average`, `min`, `max`, `stddeviation`) and functions. A blank or invalid input gives a blank result.

### Link resolver
Turns `[[names]]` into internal ids on import, and back into current names on export. It flags missing, ambiguous and circular links, and finds users and contacts the importing user has access to.

### Document writer
Turns a brainstorm model back into `.bsd` text, including its front matter. Links are written with current names, and calculated values are left out. Known elements are written in their declared order, followed by unknown elements in the order they were written. The same model always gives the same text.

### Upgrade check
Parses a document with its current structure version and with a newer one, and lists the flags that the newer version would add. This shows a user exactly what an update would break in their own document.

## Programs and repetition

### Copy engine
Creates occurrences and starts. It copies the plan (`!` and `!!` elements and all lists), locks `!!` elements, fills defaults, turns relative dates into real dates, and dates names of occurrences. Occurrences lose their schedules and starts keep them.

### Scheduler
Turns each Schedule into an iCalendar rule, works out the next dates, and asks the copy engine for an occurrence when one is due. A due date is skipped while the original's plan has problems. Times are the follower's local time.

## Brainstorm service

### Accounts
Users, pending users invited by email, and user settings such as the default import mode. Phase 1 has only the Free tier.

### Storage
Stores structures, brainstorms, contacts, copies and uploaded media for every user, free or paid, so that a user's tasks are on every device they use. Users can still keep their documents as local files, for privacy or for working with AI, and import them when they choose.

### Sign in
A long-lived cookie on desktops and tablets, and a long-lived refresh token in the installed phone app, so most people sign in once. Enterprises will set their own rules later.

### Program versions
Stores every published version of a program's structure. A program's web address with a version gets that version, and one without gets the latest. Each request checks that the user is allowed to use the program. Publishing is refused unless the structure has a new, higher version.

### Import and export
Exports any brainstorm as a document. Imports a document as a new brainstorm, in the user's chosen mode (default, referential or strict), then replaces the old brainstorm or keeps both. A refused import changes nothing and lists every problem with its line.

### My tasks
Collects every task for a user across their brainstorms and the programs they follow, so the phone app can show them in one list.

### Program publishing
Publishes a free program as a link. A program is a structure and a brainstorm with a plan. It can only be published when its plan passes the strict check and its media is uploaded. Followers start the program, but can not export it.

## Clients

### Web app
For desktops and tablets, with nothing to install.
* A "My Brainstorms" sidebar that collapses when a brainstorm opens.
* A tree of the brainstorm's sections and items on the left.
* Filters by text and tags, each belonging to the view it is in. The tree's filter keeps an item visible when it or anything below it matches. An item opened from the tree shows all its children, unless its own list of children is filtered. The kanban board has its own filter. Filtering is done in the client.
* A main view split into the list of items under the selected tree node and the selected item.
* Tabs for the main view, a kanban board of all tasks under the selected item, and the full document.
* Views generated from the structure: simple lists as plain lists, lists of complex items as grids, and a form for an item opened from a grid. This is recursive.
* A File > New menu for a new brainstorm, a new structure, or a new item under the selected one, offered only where the selected type has lists of complex items. A new brainstorm starts by choosing its structure: one of the user's structures, a program they follow, or built-in. The choice is written into the document's front matter, so the editor can help from the first line.

### Text editor
Edits `.bss` and `.bsd` documents with syntax highlighting, flags, completion, and help taken from structure descriptions. A candidate is CodeMirror. Switching between the text editor and the structured views keeps the document and the model the same.

### Program updates
When a newer version of a document's structure exists, the client downloads every version in between and shows their change notes together, newest first. It runs the upgrade check on the user's document and shows what would be flagged. The user chooses whether to update, and updating changes the version in the document's front matter.

### Phone app
An installable web app that shows "My tasks", with its own filter by text and tags, and read-only views of the brainstorms those tasks belong to, such as a risk assessment, for context. It is not for writing documents, and it does not sell programs.

## AI
Most people will get the most from AI by giving it a structure and their goals, and asking it to write a brainstorm document. The rest of this section supports that first, then lets AI work inside the system.

All AI features exchange `.bsd` text with the AI rather than a separate data format. AI writes it well, and the language core parses and validates it like any other document, so the flags it returns let the AI correct its own mistakes.

### Copy for AI (Phase 1)
A button that copies a short guide to the language, the current structure, and a starting instruction such as "help me write a brainstorm document for my goals". When an import is flagged, the problems and their line numbers can be pasted back to the AI to fix.

### Command line tool (Phase 1)
Wraps the language core, for example `brainstorm check plan.bsd` and `brainstorm tasks plan.bsd`. AI agents that work with local files can run it to check what they write. It is also how the samples are tested.

### AI tools (Phase 2)
One set of operations that both the built-in chat and the MCP server use:
* **Metadata**: the structures the user's brainstorms use, given as `.bss` documents, because they already describe every type and element in plain words.
* **Read**: task lists, kanban boards and pipelines, filtered by brainstorm, item or status, and single items or whole brainstorms, as `.bsd` text.
* **Write**: create items under a parent from `.bsd` text, and update tasks, such as their status, dates and comments. Writes are validated, and the flags are returned.

### Built-in AI chat (Phase 2, paid)
Paid tiers only, because every request costs money. Free users get the same help by giving the structure, their document and the README to any online AI chat.

A chat inside the app that helps users write brainstorm documents and structures, and create and update items in the structured view. It uses the AI tools. The app puts the guide to the language and the current structure at the start of every request and uses prompt caching, so this context is cheap to send each time.

### MCP server (Phase 2, paid)
Exposes the AI tools to the user's own AI assistant. In MCP, the metadata is offered as resources and the reads and writes as tools.

## Other editors
The language specification (README.md) is open source, so anyone can build tools for it, such as an extension for VS Code. We may build one ourselves for people who prefer to brainstorm in VS Code. It would reuse the language core through a language server, giving the same highlighting, completion and flags as the web app's editor.

## Later phases
* **Phase 2, authors**: paid programs, the Author tier, the marketplace with ratings and reviews, follower analytics, payments and payouts, the calendar feed, and the AI tools, chat and MCP server.
* **Phase 3, teams**: shared brainstorms, delegated tasks, team kanban boards and the Team tier. Native apps, if the web apps are not enough.
* **Phase 4, enterprise**: single sign-on, audit logs, private marketplaces and the Enterprise tier.
