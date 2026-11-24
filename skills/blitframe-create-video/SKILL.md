---
name: blitframe-create-video
description: Makes a new video from a description, such as a promo, ad, title card, intro, social clip or other motion graphic, by writing a small composition (data.json) and rendering it to MP4 on Blitframe. Checks what the plan allows, writes the composition from a starter with text, a shape and animation, creates a project, adds the composition in an edit, previews a frame and renders, with the Blitframe MCP tools or with curl. Use when the user asks to make, create or generate a video, animation, promo, ad or motion graphic and there is no project or data.json for it yet, whether or not they name Blitframe, or wants a quick first render.
license: MIT
---

# Create a video on Blitframe

From nothing to an MP4 link in eight steps. MCP tools are written `blitframe:<tool>`; the same flow in curl is at the end. The `blitframe` skill has the model and the safety rules.

Blitframe does not design the video. You write the composition (`data.json`), and Blitframe renders it.

## Steps

1. **Check what the workspace may render.** `blitframe:get_usage` (`GET /usage`).
   - `renderSecondsRemaining` is the balance. `limits.maxDurationSeconds`, `limits.maxLongEdge` and `limits.maxShortEdge` cap one render. The free trial renders up to 30 seconds at up to 1280×720. `limits.maxProjectBytes` caps the files of one version together (100 MB on the free trial).
   - `noTrial: true` or `planEnded: true` means the workspace cannot render or add files until a plan starts. Stop and tell the user; do not try.
   - A video costs its length in seconds, rounded up, at least 1, at any size.
2. **Pick a font from the library.** `blitframe:list_fonts` with `q` (`GET /fonts?q=mont`). Copy the font's `declaration` into `fonts.list`, and use its `fName` as the text's `f`. Never invent a font name.
3. **Write `data.json`.** Start from the composition below. For anything it does not show, read the `blitframe-data-json` skill first.
4. **Create the project.** `blitframe:create_project` with `name` (`POST /projects`). The answer's `id` is the project; its `headVersionId` is `null` until the first edit.
5. **Add the files.** `blitframe:edit_project` (`POST /projects/{projectId}/versions`) with `files: [{ "path": "data.json", "content": "<the whole file as text>" }]`. An image goes in the same edit as `{ "path": "images/logo.png", "contentBase64": "..." }`; a file over about 45 MB goes through an upload (the `blitframe` skill's `references/uploads.md`).
6. **Check the version.** `renderable` must be `true`. Otherwise fix every item in `problems` (a missing font or file, a layer with nothing to draw) and edit again with `expectedHeadVersionId` set to the version you just made.
7. **Preview one frame.** `blitframe:render` with the new version's `id` as `versionId`, `format: "frames"` and `frames: { "start": 60, "end": 61 }` (`end` is exclusive), then `blitframe:get_render` with `waitSeconds: 50`. Over MCP, look at the image it returns. Over REST, download `output.frameUrls[0]` and look at that file. Pick a frame where everything is on screen. One frame costs 1 render second.
8. **Render and hand over.** `blitframe:render` with the same `versionId` and `format: "mp4"`, then `blitframe:get_render` with `waitSeconds: 50` until `status` is `succeeded`, `failed` or `cancelled`. Once it has succeeded, give the user `output.mp4Url`. The link lasts an hour.

## A starter composition

Five seconds at 1280×720 and 30 frames a second: a dark background, a headline that fades in while it slides up, and an accent bar under it.

```json
{
  "v": "5.12.2", "nm": "Hello", "fr": 30, "ip": 0, "op": 150, "w": 1280, "h": 720, "ddd": 0,
  "assets": [],
  "fonts": { "list": [{ "fName": "Montserrat-Bold", "fFamily": "Montserrat", "fStyle": "Bold" }] },
  "layers": [
    {
      "ty": 5, "ind": 1, "nm": "Headline", "ip": 0, "op": 150, "st": 0, "sr": 1,
      "ks": {
        "a": { "a": 0, "k": [0, 0, 0] }, "s": { "a": 0, "k": [100, 100, 100] }, "r": { "a": 0, "k": 0 },
        "p": { "a": 1, "k": [
          { "t": 15, "s": [640, 400, 0], "i": { "x": 0.4, "y": 1 }, "o": { "x": 0.6, "y": 0 } },
          { "t": 45, "s": [640, 370, 0] }
        ] },
        "o": { "a": 1, "k": [
          { "t": 15, "s": [0], "i": { "x": 0.4, "y": 1 }, "o": { "x": 0.6, "y": 0 } },
          { "t": 45, "s": [100] }
        ] }
      },
      "t": { "d": { "k": [{ "t": 0, "s": { "t": "Hello", "f": "Montserrat-Bold", "s": 96, "lh": 110, "ls": 0, "tr": 0, "j": 2, "fc": [1, 1, 1] } }] }, "a": [] }
    },
    {
      "ty": 4, "ind": 2, "nm": "Accent", "ip": 0, "op": 150, "st": 0, "sr": 1,
      "ks": {
        "a": { "a": 0, "k": [0, 0, 0] }, "p": { "a": 0, "k": [640, 430, 0] }, "s": { "a": 0, "k": [100, 100, 100] },
        "r": { "a": 0, "k": 0 }, "o": { "a": 0, "k": 100 }
      },
      "shapes": [{ "ty": "gr", "nm": "Bar", "it": [
        { "ty": "rc", "nm": "Rectangle", "d": 1, "p": { "a": 0, "k": [0, 0] }, "s": { "a": 0, "k": [160, 8] }, "r": { "a": 0, "k": 4 } },
        { "ty": "fl", "nm": "Fill", "r": 1, "c": { "a": 0, "k": [1, 0.75, 0.2, 1] }, "o": { "a": 0, "k": 100 } },
        { "ty": "tr", "p": { "a": 0, "k": [0, 0] }, "a": { "a": 0, "k": [0, 0] }, "s": { "a": 0, "k": [100, 100] }, "r": { "a": 0, "k": 0 }, "o": { "a": 0, "k": 100 }, "sk": { "a": 0, "k": 0 }, "sa": { "a": 0, "k": 0 } }
      ] }]
    },
    {
      "ty": 1, "ind": 3, "nm": "Background", "ip": 0, "op": 150, "st": 0, "sr": 1, "sw": 1280, "sh": 720, "sc": "#101725",
      "ks": {
        "a": { "a": 0, "k": [0, 0, 0] }, "p": { "a": 0, "k": [0, 0, 0] }, "s": { "a": 0, "k": [100, 100, 100] },
        "r": { "a": 0, "k": 0 }, "o": { "a": 0, "k": 100 }
      }
    }
  ]
}
```

How to change it:

- **Font.** Replace the `fonts.list` entry with the `declaration` of the font you picked, and the text's `f` with its `fName`.
- **Copy.** The text is `t.d.k[0].s.t`. `s` is the font size, `lh` the line height, `j` the alignment (`0` left, `1` right, `2` center) and `fc` the fill color as RGB from 0 to 1.
- **Where text sits.** A text layer's position is its baseline, not the middle of the letters: the letters stand on `y` and rise above it. `j: 2` centers each line on `x`.
- **Where a shape sits.** A rectangle whose own `p` is `[0, 0]` is centered on its layer's position. A shape group lists its paths first, then the fill or stroke, then its `tr` transform.
- **Motion.** A keyframe is `{ "t": frame, "s": value }` with its easing in `i` and `o`; the next keyframe's `s` is where the move ends, and the last keyframe holds. Write the easing's `x` and `y` as plain numbers, on a position as on an opacity (lists with one number per component read the same). A one-number value is written `[0]` in a keyframe.
- **Length.** `op` is the frame after the last one: 150 frames at `fr` 30 is 5 seconds. Change the composition's `op` and every layer's `op` together, or the layers end early.
- **Size.** Change `w` and `h` (even numbers for MP4), the background's `sw` and `sh`, and the positions of the other layers.
- **Adding or reordering layers.** Read the `blitframe-data-json` skill first: stacking follows `ind` (smaller is in front), never the array order, and every value keeps its `{ "a": ..., "k": ... }` wrapper.

## The same flow with curl

Needs the `BLITFRAME` and `BLITFRAME_API_KEY` variables from the `blitframe` skill, and `data.json` in the current folder. The commands use `jq`, but any JSON tool builds the edit's body: the one requirement is that `content` holds the whole file as a single JSON string. Without `jq`, write the body with Node.js or Python and send it with `-d @body.json`:

```bash
node -e 'const fs = require("fs"); fs.writeFileSync("body.json", JSON.stringify({ message: "First version", files: [{ path: "data.json", content: fs.readFileSync("data.json", "utf8") }] }))'
```

```bash
auth="Authorization: Bearer $BLITFRAME_API_KEY"
json="Content-Type: application/json"

curl -s $BLITFRAME/usage -H "$auth" | jq '{renderSecondsRemaining, trial, planEnded, noTrial, limits}'

PROJECT_ID=$(curl -s -X POST $BLITFRAME/projects -H "$auth" -H "$json" -d '{"name":"Hello Blitframe"}' | jq -r .id)

VERSION=$(jq -n --rawfile composition data.json \
  '{message: "First version", files: [{path: "data.json", content: $composition}]}' |
  curl -s -X POST $BLITFRAME/projects/$PROJECT_ID/versions -H "$auth" -H "$json" -d @-)
echo "$VERSION" | jq '{id, renderable, problems, warnings}'
VERSION_ID=$(echo "$VERSION" | jq -r .id)

FRAME_ID=$(curl -s -X POST $BLITFRAME/renders -H "$auth" -H "$json" \
  -d "{\"versionId\":\"$VERSION_ID\",\"format\":\"frames\",\"frames\":{\"start\":60,\"end\":61}}" | jq -r .id)
curl -s $BLITFRAME/renders/$FRAME_ID -H "$auth" -H "Prefer: wait=50" | jq -r '.output.frameUrls[0]' | xargs curl -s -o frame.png
```

Look at `frame.png`. Queue the MP4 only when the frame is right; it uses the variables set above:

```bash
RENDER_ID=$(curl -s -X POST $BLITFRAME/renders -H "$auth" -H "$json" \
  -d "{\"versionId\":\"$VERSION_ID\",\"format\":\"mp4\"}" | jq -r .id)
curl -s $BLITFRAME/renders/$RENDER_ID -H "$auth" -H "Prefer: wait=50" | jq '{status, error, output}'
```

Repeat a `GET /renders/{renderId}` until `status` is `succeeded`, `failed` or `cancelled`.

## When it does not work

| What you see | What to do |
|---|---|
| `renderable: false` | Read `problems`. Fix each one in `data.json` or add the missing file, then edit again |
| A font named in `problems` | Use a library font (`blitframe:list_fonts`), or add the font file in an edit (the `blitframe-data-json` skill, Fonts) |
| `403 plan_limit` | The composition is longer or larger than the plan allows. Make it smaller or shorter, or tell the user what the message says |
| `403 project_too_large` | An upload, edit, ZIP import, copy or variation would make a version's files total more than the plan allows (`limits.maxProjectBytes`). A ZIP is checked at import; its upload is refused only when it is too large to import over any head. Remove or shrink files, or tell the user a larger plan allows larger projects. Do not retry |
| `402 insufficient_render_seconds` | Tell the user what the message says. Do not retry |
| `422 version_not_renderable` | The version has problems, its files total more than 8 GB, a video was asked of an odd width or height, or what the render writes alone needs more than 40 GB (choose a lower `quality` or make it shorter). The message says which |
| The render `failed` | `error.code` says why (the `blitframe-render` skill). Fix the version and render again. After `interrupted`, just render again; when its message names a missing file, add that file again with an edit first |

The full walkthrough in curl, Python, Node.js, PHP, Ruby, Go, C# and Java: `https://codeartworks.com/docs/blitframe/quickstart.md`.
