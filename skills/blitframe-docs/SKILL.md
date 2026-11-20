---
name: blitframe-docs
description: Finds and reads the Blitframe documentation. Covers the list of docs pages, fetching any page or one section of it as Markdown, llms.txt, the OpenAPI document, and the MCP tools and resources that return the same pages. Use when a Blitframe question is not answered by the other blitframe skills, when an exact field, limit, error code, endpoint or webhook event is needed, or before telling the user that Blitframe cannot do something.
license: MIT
---

# Blitframe docs

The docs are the source of truth for fields, limits and error codes. Look a fact up instead of guessing it. MCP tools are written `blitframe:<tool>`.

## How to read a page

| You want | Fetch |
|---|---|
| A page as Markdown | `https://codeartworks.com/docs/blitframe/<page>.md` |
| One section of a page | `https://codeartworks.com/docs/blitframe/<page>/<section>.md`, where `<section>` is the id of a `##` heading: its words in lower case, joined by hyphens |
| The list of pages | `https://codeartworks.com/docs/blitframe/llms.txt` |
| Every product's docs | `https://codeartworks.com/docs/llms.txt` |
| The same page for people | `https://codeartworks.com/docs/blitframe/<page>` |

Over MCP, without fetching anything:

- `blitframe:read_docs` without arguments lists every page with its section ids. With `page`, it returns that page; with `page` and `section`, one section.
- `blitframe:read_data_json_reference` returns the composition format reference: without arguments the editing rules and a section index, then one `section` at a time.
- The pages are also MCP resources: `blitframe://docs/<page>`, and `blitframe://docs/data-json-format/<section>` for each section of the format reference.

A few pages show pictures, as Markdown images with a description. An image address that starts with `/docs/` is on the same host as the page, `codeartworks.com`; the description says what the picture shows, so you rarely need to fetch it.

Prefer one section over a whole page: the format reference alone is very long. A section fetched on its own keeps the page's links, so a link such as `(#blend-modes-and-effects)` points at another section of the same page: fetch it as `<page>/blend-modes-and-effects.md`.

## Pages

| Page | What it answers |
|---|---|
| [`quickstart`](https://codeartworks.com/docs/blitframe/quickstart.md) | A first video with the API, from a sample composition to a saved MP4, in curl, Python, Node.js, PHP, Ruby, Go, C# and Java, and what to check when it fails |
| [`examples`](https://codeartworks.com/docs/blitframe/examples.md) | The whole flow as one program, in the same languages |
| [`concepts`](https://codeartworks.com/docs/blitframe/concepts.md) | The terms (project, version, head, variation, render, replacement, upload, render seconds) and how they fit together; the free trial and plans |
| [`authentication`](https://codeartworks.com/docs/blitframe/authentication.md) | API keys: creating, listing, revoking, who can revoke which |
| [`projects`](https://codeartworks.com/docs/blitframe/projects.md) | Projects, editing a project, paths and file types, undo, duplicating a project, versions, exporting a version |
| [`variations`](https://codeartworks.com/docs/blitframe/variations.md) | A second edition of a template in the same project: creating, editing, rendering and deleting a variation, its history, and when a copy or replacements fit better |
| [`uploads`](https://codeartworks.com/docs/blitframe/uploads.md) | Large files, multipart uploads, ZIP import, copying files, file capabilities |
| [`after-effects`](https://codeartworks.com/docs/blitframe/after-effects.md) | Installing Blitframe for After Effects, exporting with it or with Bodymovin (and what to add by hand then), packaging and uploading the export, checking and rendering it, tips for fast renders |
| [`after-effects-support`](https://codeartworks.com/docs/blitframe/after-effects-support.md) | Which After Effects features render: layers, transforms, masks, shapes, blend modes, text, expressions, layer styles, effects; the media files a template can use; what the Validator list gets wrong |
| [`data-json`](https://codeartworks.com/docs/blitframe/data-json.md) | A complete small composition, and what Blitframe checks |
| [`data-json-format`](https://codeartworks.com/docs/blitframe/data-json-format.md) | The full composition format reference |
| [`fonts`](https://codeartworks.com/docs/blitframe/fonts.md) | Project fonts, the font library, license confirmation |
| [`editing`](https://codeartworks.com/docs/blitframe/editing.md) | Replacing text, fonts, images and video by layer, in an edit or a render; many videos from a data sheet |
| [`text-fitting`](https://codeartworks.com/docs/blitframe/text-fitting.md) | Keeping replaced copy inside its space |
| [`renders`](https://codeartworks.com/docs/blitframe/renders.md) | Formats, quality, status, output, batch renders, archives, HTML5 ads, pricing, plan limits |
| [`sharing`](https://codeartworks.com/docs/blitframe/sharing.md) | Share links |
| [`webhooks`](https://codeartworks.com/docs/blitframe/webhooks.md) | Events, delivery, signatures, the delivery log |
| [`errors`](https://codeartworks.com/docs/blitframe/errors.md) | Every error code and what to do, and what is safe to retry |
| [`limits`](https://codeartworks.com/docs/blitframe/limits.md) | Sizes, durations, counts, rate limits, plan limits, the render-second balance, retention |
| [`api`](https://codeartworks.com/docs/blitframe/api.md) | Every REST endpoint, the conventions they share, and retrying safely with an `Idempotency-Key` |
| [`mcp`](https://codeartworks.com/docs/blitframe/mcp.md) | Connecting an agent, signing in, results and errors, protocol details |
| [`agent-reference`](https://codeartworks.com/docs/blitframe/agent-reference.md) | Every MCP tool, and the rules for using them: safety, workflow, editing, rendering, cost, retries, uploads, composition rules, errors |
| [`ai-agents`](https://codeartworks.com/docs/blitframe/ai-agents.md) | Setting up a coding agent: one prompt that makes a first video, the MCP server, installing the skills and the plugin |

Sections that are asked for often:

- Pricing and refunds: `https://codeartworks.com/docs/blitframe/renders/pricing.md`
- Plan limits: `https://codeartworks.com/docs/blitframe/limits/plan-limits.md`
- Render-second balance (`GET /v1/usage`): `https://codeartworks.com/docs/blitframe/limits/balance.md`
- Retrying safely (idempotency keys): `https://codeartworks.com/docs/blitframe/api/retrying-safely.md`
- Variations: `https://codeartworks.com/docs/blitframe/variations.md`
- Webhook events: `https://codeartworks.com/docs/blitframe/webhooks/events.md`
- Verifying webhook signatures: `https://codeartworks.com/docs/blitframe/webhooks/verifying-signatures.md`
- Editing a project: `https://codeartworks.com/docs/blitframe/projects/editing-a-project.md`
- Text and fonts in `data.json`: `https://codeartworks.com/docs/blitframe/data-json-format/text-and-fonts.md`

## The REST API as a schema

- OpenAPI 3.1: `https://blitframe.codeartworks.com/openapi/v1.json` (also at `https://codeartworks.com/docs/openapi/blitframe.json`).
- A page for every endpoint, with its fields and a request in curl, Python, Node.js, PHP, Ruby, Go, C# and Java: `https://codeartworks.com/reference/blitframe`.
- Over MCP, each tool's input schema is the request's shape, and a tool runs the same operation as its REST endpoint.

## The account, not the product

Workspaces, roles and signing in are the same for every Code Artworks product: `https://codeartworks.com/docs/blitframe/workspaces.md` and `https://codeartworks.com/docs/blitframe/account.md`; connecting agents is `https://codeartworks.com/docs/blitframe/mcp.md`. Plans and prices are at `https://codeartworks.com/pricing`; billing, the team and API keys are managed in the console at `https://app.codeartworks.com/blitframe/`.

## If the docs do not answer it

Say so, and say what you checked. Do not present a guess about Blitframe as a fact. For a render or version that misbehaves, `blitframe:report_problem` (`POST /problem-reports`) reaches Blitframe support.
