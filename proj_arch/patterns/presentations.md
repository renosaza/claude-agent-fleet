# Pattern: presentations

## Rule
**Grain Deck System is the default design for every presentation / slide deck the fleet
produces.** When a project produces slides, it inherits this look — it does NOT invent its own.
Reuse the two design knobs only: `accent` (one of four colors) and `cornerStyle` (Sharp → `0px`
radius, Soft → `12px`). Everything else — fonts, palette, slide frame, the seven layouts, the
five principles — is fixed canon so every deck across the fleet reads as one system. The style is
self-contained in `proj_arch` (`templates/presentation.html.tmpl`); the scaffolder copies it into
a new project that may produce decks.

> **Note — default look, not a forced deliverable.** This canon does not decide *whether* a
> project produces a deck (that is the project's own concern); it decides *how the deck looks when
> one is produced*. Unlike PDF, slides are not gated behind an explicit request — but the Grain
> look is the default the moment slides exist. A project switches to a different deck look ONLY on
> the user's explicit instruction, exactly as a user may name a different PDF style.

> **Boundary vs. [pdf-reports.md](pdf-reports.md) — a report is not a deck.** A PDF report is
> flowing long-form prose, authored in Markdown and rendered to PDF by the pandoc fleet style
> (opt-in, explicit request only). A slide deck is 16:9 slides authored in HTML and governed
> here. They are different deliverables with different canons; neither replaces the other. If a
> deck must become a PDF (print/export), it is still authored in Grain HTML first and the Grain
> look wins — do NOT route a deck through the pandoc report style.

## When it applies
- Any project output that is a presentation, deck, or set of slides. Author it in Grain.
- A new project whose deliverables may include slides. The scaffolder copies
  `templates/presentation.html.tmpl` and wires the default-look line into the project `CLAUDE.md`.

It does NOT apply to: PDF/prose reports (see [pdf-reports.md](pdf-reports.md)); a project the user
has explicitly told to use a different slide look; non-slide HTML (dashboards, web pages).

**Authoring vs. content language.** The system, its tokens, and the template are authored in
**English** (canon invariant — see [mcp-and-skills.md](mcp-and-skills.md)). The slide *content* a
project fills in stays in that project's target language (Russian for the current fleet).

## How
**Type pairing — Google Fonts.** Schibsted Grotesk (display, headings, body; weights 400–800)
paired with Space Mono (eyebrows, labels, page numbers, metadata; weights 400/700). One link,
fixed:

```html
<link href="https://fonts.googleapis.com/css2?family=Schibsted+Grotesk:wght@400;500;600;700;800&family=Space+Mono:wght@400;700&display=swap" rel="stylesheet">
```

**Palette — light (default).**

| Token | Hex | Use |
|---|---|---|
| Paper | `#F4F2EE` | page ground |
| Surface | `#FFFFFF` | slide frame fill |
| Ink | `#17161B` | headings |
| Muted | `#76747C` | mono labels, footer text |
| Line | `#E4E1DB` | 1px borders, rules |
| Body | `#3A3942` / `#56545C` | body copy (primary / secondary) |
| Image fill | `#ECEAE4` | image placeholder ground |

**Palette — dark.**

| Token | Hex | Use |
|---|---|---|
| Base | `#131217` | page / slide ground |
| Surface | `#1C1B23` | raised fill, image ground |
| Ink | `#F1F0EC` | headings |
| Muted | `#8E8C97` | mono labels, footer text |
| Line | `#2A2932` | 1px borders, rules |
| Body | `#C9C8CF` | body copy |

**Accent — exactly one, used once per slide.** Pick one and reuse it everywhere via the
`--accent` CSS variable. Options:

| Name | Hex |
|---|---|
| Violet (default) | `#6B4FE8` |
| Blue | `#2D6BE0` |
| Orange | `#E8572A` |
| Green | `#1F8A5B` |

**Corner radius — `cornerStyle`.** Sharp → `--radius: 0px` (default); Soft → `--radius: 12px`.
One choice per deck, applied to every frame/box.

**Slide frame.** `aspect-ratio:16/9`; padding `56px 62px`; 1px Line border; `border-radius`
= `--radius`; Surface fill. Vertical structure, top to bottom: a Space Mono eyebrow (uppercase,
`letter-spacing:0.2em`, accent or Muted) → the slide content → a footer (Line top-border,
`padding-top:18px`) with, on the left, an accent square + "Grain Deck System" in mono and, on the
right, the page number `NN / 24` in mono.

**The seven layouts** (each works in light and dark; the only swaps are ground/Ink/Line/body and
the footer Muted):

| Layout | One-line description |
|---|---|
| Title | Mono eyebrow, one large 700-weight headline (`~60px`), footer with accent square. |
| Section divider | A giant accent section number (`~150px`, weight 800) over a 600-weight section title. |
| Bulleted content | A 600 headline, then lines led by **square accent markers** (not bullets). |
| Two-column / comparison | Two columns; the "after" column gets the **accent** top-border, "before" gets Line. |
| Big statement | One oversized statement (`~58px`), centered in the frame, one accent word allowed. |
| Data / stat callout | Three-up giant accent numbers (`~78px`, weight 800) each over a mono caption. |
| Image-led | Image fills most of the frame; minimal type — one title and one line of context. |

## The five principles (do / don't)
1. **One accent, used once.** A single hue carries every emphasis; at most one accent moment per
   slide. — *Don't* spread the accent across several elements on one slide.
2. **Type does the work.** Hierarchy comes from size and weight. — *Don't* build hierarchy from
   boxes, fills, or color.
3. **Whitespace over decoration.** Generous margins replace borders and shadows. — *Don't* add
   drop-shadows, gradients, or filler graphics.
4. **Mono labels for structure.** Space Mono tags eyebrows, page numbers, metadata. — *Don't* set
   labels in the body face.
5. **Square accent markers replace bullets.** A small accent square marks each point. — *Don't*
   use real `•`/`<ul>` bullets or icons.

## Anti-patterns
- **A per-project bespoke slide look** → every deck inherits Grain; only `accent` and
  `cornerStyle` vary. A different look requires the user's explicit instruction.
- **More than one accent (or accent color) per slide** → one hue, one moment per slide.
- **Decoration over whitespace** → no shadows, boxes, gradients, or chrome standing in for
  margins.
- **Body font for labels** → eyebrows / page numbers / metadata are always Space Mono.
- **Real bullets instead of square markers** → square accent markers, never `•` or list icons.
- **Routing a deck through the PDF report style** → a deck is Grain HTML even when exported to
  PDF; the pandoc style in [pdf-reports.md](pdf-reports.md) is for prose reports only.
- **Hardcoding a personal path** → the style lives in `proj_arch/templates`; never reference a
  `Downloads` folder or external kit path in any pattern, agent, or project file.

## Minimal example
One Title slide (light, Sharp corners, default violet accent). `--accent` / `--radius` are set on
a wrapper; the slide content is in the project's target language.

```html
<div style="--accent:#6B4FE8; --radius:0px; font-family:'Schibsted Grotesk',system-ui,sans-serif;">
  <div style="background:#FFFFFF; border:1px solid #E4E1DB; border-radius:var(--radius); padding:56px 62px; aspect-ratio:16/9; display:flex; flex-direction:column; justify-content:space-between; color:#17161B;">
    <div style="font-family:'Space Mono',monospace; font-size:13px; letter-spacing:0.2em; text-transform:uppercase; color:var(--accent);">Annual Review — 2026</div>
    <h2 style="font-size:60px; font-weight:700; line-height:1.03; letter-spacing:-0.03em; max-width:80%; margin:0;">Building quiet software for noisy work.</h2>
    <div style="display:flex; justify-content:space-between; align-items:center; border-top:1px solid #E4E1DB; padding-top:18px;">
      <div style="display:flex; align-items:center; gap:12px;"><span style="width:12px; height:12px; background:var(--accent); display:block;"></span><span style="font-family:'Space Mono',monospace; font-size:11px; letter-spacing:0.16em; text-transform:uppercase; color:#76747C;">Grain Deck System</span></div>
      <span style="font-family:'Space Mono',monospace; font-size:11px; letter-spacing:0.16em; color:#76747C;">01 / 24</span>
    </div>
  </div>
</div>
```

The full skeleton — Google Fonts link, tokens as CSS custom properties, and all seven layouts
ready to fill (light, with a documented dark switch) — is `templates/presentation.html.tmpl`.
