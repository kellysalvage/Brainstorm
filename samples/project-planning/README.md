# Project planning

Build a simple project planner from scratch: a project, broken down into sub-projects and tickets, with sub-tickets, dependencies and comments. The tickets appear on the kanban board as soon as they exist, so there is a working planner from step 2 onwards.

This series is for **authors**: people who want to design their own structures. Each step adds one idea to the structure (`.bss`) and uses it in a brainstorm document (`.bsd`). Each step's files are complete, so you can open any step on its own, and comparing two steps shows exactly what changed.

Video: coming soon.

## Steps

| Step | Files | What you build | What you learn |
|------|-------|----------------|----------------|
| 1 | `01-project` | A project with a name, description and dates | Declaring a type, elements, shorthand, a condition |
| 2 | `02-tickets` | Tickets on a kanban board | Extending a built-in type, lists |
| 3 | `03-sub-projects` | Sub-projects inside sub-projects | A list that holds its own type |
| 4 | `04-sub-tickets` | Sub-tickets for big tickets | Saying that a list from the original type holds the extended type |
| 5 | `05-dependencies` | Tickets that wait for other tickets | Links and `[[names]]` |
| 6 | `06-comments` | Comments on everything, and a project owner | The built-in Comment type, and the `me` default |
| 7 | `07-template` | A reusable website launch template | Templated `!` elements and relative dates |

## Following along

For each step:
1. Import the step's `.bss` file as a structure.
2. Import the step's `.bsd` file. It names the step's structure in its front matter.
3. Open the brainstorm, and look at the tree, the items and the kanban board.

When you import a later step, choose to replace the brainstorm from the step before.

The full language is described in the [README](../../README.md) at the top of this repository.
