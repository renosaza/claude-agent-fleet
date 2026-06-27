# mathematik

Photo → Russian-language solutions PDF for university-level math.

## How to use
Either way works (or both at once for the same homework):

- Drop photos at the top of `input/` and say «реши задания», OR
- Attach photos directly in the Claude Code chat and say «реши задания».

Mathematik creates the next `input/<N>/` batch folder, moves every new file you provided
into it, and produces `output/B<N>-<timestamp>.pdf`. Past batches stay archived under
`input/1/`, `input/2/`, … so you can always rebuild or audit a previous run.

## Stack
- Vision: multimodal Claude reads images directly (no external OCR). Pages are
  auto-classified as problems / theory / mixed.
- Reasoning: opus-tier Claude solves and explains in Russian, leaning on whatever
  theoretical material (lecture notes, formula sheets, methodicals) the user puts into
  the batch.
- Typesetting: HTML + MathJax rendered by `cloakbrowser` (bundled Chromium) → PDF.
- Figures (analytic geometry, conics, vectors, 3D): JSXGraph from CDN; the solver emits
  a closed-set JSON DSL, the composer translates it to JSXGraph calls in-page.

## Requirements
- Claude Code with the `cloakbrowser` MCP server available (bundled Chromium, no system
  Chrome / TeX needed).
- Network access for MathJax + JSXGraph CDN on first render.

## Architecture
Orchestrator (`CLAUDE.md`) dispatches three workers (`.claude/agents/`):
- `vision-extractor` — photos → `tasks.json`
- `math-solver` — tasks → `solutions.json`
- `pdf-composer` — solutions → `output/<run-id>.pdf`

See `CLAUDE.md` for the full workflow.
