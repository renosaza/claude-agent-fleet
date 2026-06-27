# Pattern: pdf-reports

## Rule
PDF is **opt-in**, never the default. A project's default output format does not change: where
the default is Markdown or another format and PDF is **not** specified, a PDF is produced ONLY
on the user's **explicit** request. When a PDF report *is* requested (and the user did not name
a different style), it MUST be built in the **fleet style** — pandoc with a xelatex-compatible
engine and the defaults below — so every general report across the fleet looks the same. The
style is self-contained in
`proj_arch` (`templates/pandoc-pdf.yaml.tmpl`, `templates/md2pdf.sh.tmpl`); it is localizable to
the project's target language.

> **Note — this canon covers general text/prose reports only.** Domain renderers that already
> produce PDF their own way stay as they are: `mathematik/pdf-composer` renders via
> `cloakbrowser` + `cloak_pdf` with MathJax (NO LaTeX, because the host has no TeX distribution)
> for formula-heavy content; `rnd` ships `report.md` + `report.html` (chart.js) by default. The
> pandoc fleet style is for general prose reports, not a replacement for those.

## When it applies
- The user explicitly asks for a PDF report of general text/prose content (no special style
  named). Apply the fleet style.
- A new project whose deliverable may be exported to PDF on request. The scaffolder copies the
  two templates and wires the opt-in line into the project `CLAUDE.md` hard rules.

It does NOT apply when: PDF was not requested (keep the project's default format); the user named
a different style; or the content is formula-heavy math handled by `mathematik/pdf-composer`.

## How
The fleet style is **pandoc** with a **xelatex-compatible** engine. `xelatex` is the conceptual
standard; the installed, drop-in engine on this host is **tectonic** (`pdf-engine: tectonic` —
there is no `xelatex` command). A `pdf-engine` set in the defaults file OVERRIDES the
`--pdf-engine` CLI flag, so the real engine must be correct inside the yaml. Build a PDF from a
Markdown source with the shared defaults file:

```
pandoc report.md -o report.pdf -d pandoc-pdf.yaml
# or via the wrapper (yaml resolved relative to the script):
./md2pdf.sh report.md [report.pdf]   # OUT defaults to IN with a .pdf extension
```

**Style settings** (the defaults file): `pdf-engine: tectonic`, `toc: true`, `toc-depth: 3`,
`number-sections: false`, `mainfont: "PT Serif"`, `monofont: "Menlo"`,
`fontsize: 11pt`, `geometry: margin=2cm`, `linkcolor: blue`, `urlcolor: blue`,
`toc-title: "Содержание"`.

**Localization.** Two settings are localized per project: `toc-title` and the fonts. The default
`toc-title` is `"Содержание"` because the current fleet is Russian-facing; a project in another
target language sets its own (e.g. `"Contents"`). Keep the rest of the style fixed.

**Font fallback.** `PT Serif` / `Menlo` are macOS system fonts present out of the box; `PT Serif`
is the verified-working default for Cyrillic. `DejaVu Serif` / `DejaVu Sans Mono` also carry
Cyrillic but are not installed by default — opt in only after installing them. Other system fonts
with Cyrillic coverage for `mainfont`: `"Georgia"`, `"Times New Roman"`, `"Helvetica Neue"`.

**Dependency install (one-time, separate step).** The host needs pandoc and a xelatex-compatible
engine. On macOS, install pandoc and tectonic via Homebrew:

```
brew install pandoc tectonic
```

A full MacTeX distribution (`brew install --cask mactex-no-gui`, which provides a real `xelatex`)
is a heavier alternative if you prefer it. Do not block report authoring on the install: write
the Markdown regardless; the install is handled once, out of band, and the build is the final step.

## Anti-patterns
- **Silently changing the default output to PDF** → the default format stays; build a PDF only
  on the user's explicit request.
- **Hardcoding a personal download path** → the style lives in `proj_arch` templates; never
  reference a `Downloads` folder or an external kit path in any agent, pattern, or project file.
- **Breaking a domain renderer** → do not route `mathematik` formula PDFs through pandoc/LaTeX
  (the host has no TeX dist for that pipeline by design), and do not replace `rnd`'s
  `report.html` (chart.js) default with a PDF. This canon adds an opt-in path; it overrides
  nothing existing.
- **Asserting a font or engine exists** → DejaVu and a real `xelatex` command are absent on this
  host; default to `PT Serif` + `tectonic`, and fall back rather than failing the build.

## Minimal example
`pandoc-pdf.yaml` (fleet style, Russian default):

```yaml
pdf-engine: tectonic
toc: true
toc-depth: 3
number-sections: false
variables:
  mainfont: "PT Serif"
  monofont: "Menlo"
  fontsize: 11pt
  geometry: margin=2cm
  linkcolor: blue
  urlcolor: blue
  toc-title: "Содержание"
```

Build on explicit request:

```
pandoc report.md -o report.pdf -d pandoc-pdf.yaml
```
