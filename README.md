# Blitframe skills

Agent Skills and plugin manifests for [Blitframe](https://codeartworks.com) by Code Artworks: the hosted GPU render API that turns Lottie compositions (the `data.json` that After Effects exports with Blitframe for After Effects or Bodymovin) into MP4, HLS, image frames and HTML5 ads.

With these installed, a coding agent such as Claude Code, Codex or Cursor knows how Blitframe works and can create projects, write and edit compositions, make many videos from one template and render videos for you.

- Docs: <https://codeartworks.com/docs/blitframe/ai-agents>
- MCP server: `https://blitframe.codeartworks.com/mcp`
- REST API: `https://blitframe.codeartworks.com/v1`

## Install

### Any agent: the skills

In your project folder:

```bash
npx skills add CodeArtworks/blitframe-skills
```

The installer asks which skills to add and for which agents. Add `--skill blitframe-render` for one skill, `-g` for every project on your machine. The same skills are published at `https://codeartworks.com`:

```bash
npx skills add https://codeartworks.com
```

Then connect the MCP server so the agent has tools as well as knowledge: <https://codeartworks.com/docs/blitframe/mcp> has the setup for each agent. Without it, the skills use the REST API with curl and the key in `BLITFRAME_API_KEY`.

### Claude Code: the plugin

The plugin installs the skills and connects the MCP server.

```bash
claude plugin marketplace add CodeArtworks/blitframe-skills
claude plugin install blitframe@codeartworks
```

Start Claude Code, run `/mcp`, choose `blitframe` and sign in with your Code Artworks account. No API key is needed.

To use an API key instead, skip the plugin, install the skills as above, and add the server with a header:

```bash
claude mcp add --transport http blitframe https://blitframe.codeartworks.com/mcp \
  --header "Authorization: Bearer $BLITFRAME_API_KEY"
```

### Codex, Cursor and GitHub Copilot

This repository carries the manifests these agents read: the portable [Agent Plugins](https://agent-plugins.org) files (`plugin.json` and `mcp.json`), `.codex-plugin/plugin.json` and `.cursor-plugin/plugin.json`. Until the plugin is listed in their directories, install the skills with `npx skills add CodeArtworks/blitframe-skills` and connect the MCP server as <https://codeartworks.com/docs/blitframe/mcp> describes.

## First prompt

With an API key in `BLITFRAME_API_KEY` (create one in the [console](https://app.codeartworks.com/blitframe/)):

```text
Make a 5-second 1280x720 video with Blitframe: the word "Hello" in white on a dark blue
background, fading in. Render one frame and look at it, then render an MP4 and give me
the link.
```

## The skills

| Skill | What it teaches |
|---|---|
| [`blitframe`](skills/blitframe/SKILL.md) | Start here: projects, versions and renders, editing and rendering, safety rules, uploads, errors, MCP tools or REST |
| [`blitframe-create-video`](skills/blitframe-create-video/SKILL.md) | From nothing to a first MP4, with a starter composition |
| [`blitframe-data-json`](skills/blitframe-data-json/SKILL.md) | Writing and editing `data.json`, and reading the format reference one section at a time |
| [`blitframe-variations`](skills/blitframe-variations/SKILL.md) | Variations of a template: a lasting variation per language or brand, replacements in a render, one batch from a spreadsheet, a separate copy, text fitting |
| [`blitframe-render`](skills/blitframe-render/SKILL.md) | Formats, quality, cost, batch renders, ZIP archives, share links, HTML5 ads |
| [`blitframe-docs`](skills/blitframe-docs/SKILL.md) | Finding any docs page and fetching it as Markdown |

The skills are Markdown only. They contain no scripts and run no code on your machine.

## What is in this repository

| Path | For |
|---|---|
| `skills/` | The skills, in the [Agent Skills](https://agentskills.io) format |
| `plugin.json`, `mcp.json` | Agent Plugins 1.0: the plugin and its MCP server |
| `.claude-plugin/`, `.mcp.json` | Claude Code: the plugin, its marketplace (`codeartworks`) and its MCP server |
| `.codex-plugin/`, `.cursor-plugin/` | Codex and Cursor manifests |

## Support

Docs for people and agents: <https://codeartworks.com/docs/blitframe/> (every page is also Markdown at `/docs/blitframe/<page>.md`, listed in <https://codeartworks.com/docs/blitframe/llms.txt>). Questions and problems: <https://codeartworks.com/#contact>.

## License

[MIT](LICENSE)
