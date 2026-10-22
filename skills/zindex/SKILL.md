---
name: zindex
description: Immutable agent diagrams. Save, change and render diagrams in Zindex so they outlast one conversation. Use when the user wants a diagram they can come back to, change later from another tool, compare over time or see as it was on a date; when a repository has a .zindex.json file; or when the diagram_* tools are available.
---

# Zindex

Immutable agent diagrams. Every saved version is kept exactly as it was, with its own revision number and time, so a diagram made today can be read, changed and rendered the same way later, from any MCP client or the REST API.

## The loop

1. **Find or create.** `diagram_find` lists the workspace's diagrams. `diagram_create` saves a new one from a DSP scene, or from Mermaid text (`erDiagram` or `flowchart`), whose node IDs become the element IDs. Give it a `key` such as `payments-architecture`: repeating the create returns the same diagram instead of a copy.
2. **Read.** `diagram_get` returns a compact outline: a header, the brief, the last three changes, then one element per line. Pass `knownRevision` with the revision you last saw to get only what changed since. `view: "mermaid"` returns the diagram as Mermaid instead (not for sequence diagrams).
3. **Change.** `diagram_edit` with `baseRevision` (the revision you last read), a one-line `message` and `ops`, or `mermaid`: the whole diagram as Mermaid, edited from `view: "mermaid"`. Zindex applies only the differences, so unchanged elements keep their IDs. In a sequence diagram, `after` or `before` on a create or update op puts a lifeline, message or note between two others. On `REVISION_CONFLICT` the error lists what changed; re-plan against the new revision. A `requestId` makes a retry after a timeout return the first result instead of applying twice. Any call that draws can answer `BUSY`: wait `retryAfter` seconds and send it again. `DRAW_TIMEOUT` means a save or edit saved nothing, so don't resend the same scene: draw fewer edges into one node, or split the diagram.
4. **Show.** `diagram_render` returns links. Give the person the SVG link: a published diagram's links don't expire but stop if it is made private or deleted (`url` is the version's page); anything else's work for 7 days, so for a README, issue or doc that stays up, publish or make a share link (the person confirms) and see https://zindex.ai/docs/api/#putting-a-diagram-in-a-doc-issue-or-readme. Ask for `preview: true` only when you need to look at the picture yourself. Never hand-edit a rendered SVG or PNG: change the diagram with `diagram_edit` and render again, or the change is in no version. A saved version's image prints its address in the bottom-right corner, `zindex.ai/d/<id>@<revision> · <date>`. Older images print `<id> | Rev N - <date>`. Shown an image with either line, call `diagram_get` with its id and `revision` to open the version it shows; a diagram in another workspace answers `NOT_FOUND`, and a published one reads with no key at `GET /v1/public/diagrams/{id}@{revision}`. An unsaved preview has none.

## Keep context with the diagram

- **Brief:** up to 1,000 characters on what the diagram shows, the conventions to keep and the decisions made. Read it before editing, and update it with `diagram_edit` (`brief`) when a convention changes.
- **Message:** one line per edit saying why.
- **Pointer file:** in a code repository, keep `.zindex.json` at the root, for example `{ "diagrams": { "architecture": { "id": "dgm_7Q2kD9a1B2c3", "revision": 12 } } }`. Read it at the start of a session and pass the revision as `knownRevision`; update it after each edit.
- **Handing off to a teammate's agent:** commit `.zindex.json` with the code. Their agent finds the diagram only if its key or sign-in acts in the team workspace that holds it; a diagram in your personal workspace answers `NOT_FOUND` to them until it is moved into the team workspace from the dashboard. They read the brief first, plan against the revision they read, and pass it as `baseRevision`. `diagram_history` shows who saved each version (`by`) and with which client.

## Workspaces

Each key or sign-in works in one workspace: the person's own, or a team workspace. Every tool acts on that workspace's diagrams, and everyone in a team workspace can read all of them. No tool takes a workspace: a diagram in another workspace answers `NOT_FOUND`, and to work in two the person connects once for each.

`READ_ONLY` means the key or sign-in can read diagrams but not change or share them or report a problem with them. Don't retry. With `reason: "credential"`, the person connects again and chooses read and write, or gives a key with write access. With `reason: "role"`, they are a viewer, and a workspace owner has to make them a member. `SHARING_DISABLED` means the workspace's owners turned public sharing off: `publish` and `link` are refused, while `unpublish` and `unlink` still work. No tool creates workspaces, invites people, changes roles or manages billing; the person does that in the dashboard, where team workspaces are a beta whose features may change. In a team workspace, a teammate's diagram with your key answers `KEY_EXISTS` to your create: read and edit it instead.

## Writing scenes

- Declare `diagramFamily` and `layoutStrategy` at the document root, and leave `layout` off nodes. The engine positions nodes, routes edges and places labels.
- Nodes keep their positions after the first save unless a node that grows needs the room: it pushes the nodes in its way (not in sequence diagrams), and the edit's `changed` lists them. An edit that adds or deletes a frame round placed nodes also places them again, listed in `changed`, when keeping them would leave frames overlapping or a frame holding a node that isn't its member. New nodes are placed next to their neighbours; edges and frames are redrawn each time; sequence diagrams follow message order. Send `autoLayout` to lay out again: `{ "op": "autoLayout" }` for every node, or with `elements` listing the nodes to place again. Laying out every node again drops a frame's own `layout` (except a pool's, a lane's or a fragment's, and in org charts and networks) and lays the frame out with its members. A frame that keeps its `layout` keeps its members where they are; name them in `elements` to place them again inside it. One that doesn't fit lands outside, reported as `NODE_OUTSIDE_FRAME`.
- Use stable, meaningful element IDs such as `api-gateway` or `orders-db`. Ops refer to them and the history tracks them. IDs show in the SVG, as labels do.
- `diagram_create` and `diagram_edit` return `warnings` for the version they save, each with a code, the element when there is one, and a one-line fix. A diagram with warnings is saved; fix them with an edit. A write with errors saves nothing and returns `VALIDATION_FAILED` with `errors`. That includes an op on something it can't change, such as `moveElement` on an edge or `resizeElement` on a node the engine hasn't placed yet, and an edit whose result would break a rule the diagram didn't. Fix what each error names and resend with the same `baseRevision`. Within one edit, an op may name an element a later op creates, except the element `after` or `before` names, which must exist when the op runs. A saved edit's warnings also note what it changed besides what its ops named, such as the constraints a `deleteElement` removed. `diagram_validate` checks a draft without saving. When the docs and the engine disagree, or something you needed is missing, tell Zindex with `diagram_share` and `action: "feedback"`: no diagram is needed, and the person confirms it first. Every code: https://zindex.ai/docs/reference/validation-rules/
- The authoring guide, [authoring.md](authoring.md) next to this file, covers element kinds, families, constraints, styles and icons. It is also at https://zindex.ai/skill/authoring.md
- Schemas: https://zindex.ai/schemas/scene.schema.json and https://zindex.ai/schemas/ops.schema.json

## Going back

- **Any version:** `diagram_get` with `revision`, or `asOf` (a date such as `2026-09-26`, meaning the end of that day in UTC, or an ISO timestamp with an offset).
- **What changed:** `diagram_history` lists versions with time, who saved each (`by`), client and message; `compare: { from, to }` lists the elements added, changed and removed between two versions or dates.
- **Undo:** `diagram_edit` with `restoreRevision` makes a new revision equal to an old one. History is never rewritten.
- **Check nothing changed:** writes, renders and `diagram_history` carry `chain`, a hash covering the version and every one before it. The same `chain` later means nothing up to it changed: https://zindex.ai/docs/concepts/#verifying

## Ask first

`diagram_delete` removes a diagram; `diagram_share` with `publish` makes it readable by anyone with its link, with `link` makes a share link that shows its images (never its scene) to anyone who has it until `unlink` revokes it, or Zindex does if the diagram breaks the content policy (on Pro and Team; on Free it returns `FEATURE_UNAVAILABLE`; publishing works on every plan and makes the scene public too), and with `report` sends one version to Zindex support (anonymized unless `anonymize` is false). The person confirms each of them. Unpublishing leaves share links working; `unlink` revokes them (what each choice shows: https://zindex.ai/docs/concepts/#sharing). Taking a link down stops new fetches at once; copies people already saved stay with them. If a tool returns `CONFIRMATION_REQUIRED`, tell the person what will happen and call again with the `confirm` token only after they agree.

## What Zindex does not do

- It renders the scene it is given. It does not draw from a picture or a description: you write the scene or the operations.
- Mermaid is accepted for erDiagram and flowchart (graph) diagrams only. Subgraphs, edges to them and styles are left out and listed in the warnings; other Mermaid types are refused.
- A Mermaid edit changes nodes and edges, and deletes the ones you leave out. Frames, groups and text keep their state; change them with ops. Mermaid edits work on architecture and entity-relationship diagrams.
- A deleted diagram can be restored for 24 hours with `diagram_delete` and `undo: true`. After that it is gone with all its versions.
- No tool exports a diagram's whole history or the whole workspace, because a tool's result goes into your context and a whole history can run to megabytes. The person downloads one diagram's history from its page on the dashboard, and every diagram from Settings → Account (https://zindex.ai/settings/account). With an API key: `GET https://api.zindex.ai/v1/diagrams/{id}/export` or `GET https://api.zindex.ai/v1/workspace/export` (a team workspace's owners only).
