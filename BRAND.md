# Brand

Creator Studio uses a small brand system (the mark system, colours, type and layout below) built so related
instruction-led workflow repos can share it, each with **one accent colour** and **one mark topology** of its own.
This page holds the exact values.

## This repo

| | |
|---|---|
| Name | **Creator Studio** |
| Accent | Ember `#FF6A3D` (6.6:1 on ink) |
| Mark topology | Fan-In: assemble many into one |
| Tagline | Plain words in. Finished media out. |
| Stage labels (banner) | Script, Voice, Music, Pictures, Video → Finished File |

Files in `assets/brand/`:

| File | Use |
|---|---|
| `mark.svg` | Square tile icon (512 viewBox 64). Avatars, docs, plugin listings. |
| `banner.svg` | 1280 x 320 README header. |
| `social-preview.svg` | 1280 x 640 source for the GitHub social preview. Export to PNG and upload under **Settings > Social preview** (GitHub does not read it from the repo, and PNGs are git-ignored here). |
| `how-it-works.svg` | 1280 x 440 diagram for the README "How it works" section. Same ink canvas, dot grid, outlined type and single accent as the banner. |

## The mark

A mark shows a **workflow shape** on a 64 x 64 grid: small paper-coloured nodes for the inputs or outputs, one larger
node in the repo accent for the result. Pick the topology that matches the repo's job.

| Topology | Meaning | Used by |
|---|---|---|
| Fan-in | assemble many stages into one result | Creator Studio |
| Fan-out | distribute one result to many outputs | reserved |
| Line | transform step by step | reserved |
| Loop | iterate and monitor | reserved |

Geometry (all in the 64-unit box; tile is `rect` 64 x 64, `rx` 14, fill ink):

- Stroke 3, round caps and joins, colour paper at 90% opacity, no fill.
- Input or output nodes: `r` 3.6, paper. End node: `r` 8.5, repo accent. Exactly one accent node.
- Fan-in paths: `M14 16C28 16 30 32 38.5 32M14 32H38.5M14 48C28 48 30 32 38.5 32`, nodes at x 14 (y 16, 32, 48), end node at (47, 32).
- Fan-out is the horizontal mirror: `M50 16C36 16 34 32 25.5 32M50 32H25.5M50 48C36 48 34 32 25.5 32`, end node at (17, 32).
- The tile stays dark in light mode. Do not recolour the paper strokes.

## Colour

| Token | Hex | Role |
|---|---|---|
| Ink | `#0E1116` | Canvas, tile |
| Surface | `#161B22` | Cards, pills |
| Line | `#2A313C` | Rules, borders |
| Paper | `#F6F4EF` | Text on dark, mark strokes (17.2:1 on ink) |
| Mute | `#9AA3B2` | Secondary text (7.4:1 on ink) |
| Mute 2 | `#5B6573` | Connectors and decoration only (3.2:1, never text) |

Accents (one per repo, at most 10% of any image):

| Accent | Hex | On ink | Assigned to |
|---|---|---|---|
| Ember | `#FF6A3D` | 6.7:1 | Creator Studio |
| Tide | `#22C7B8` | 8.9:1 | reserved |
| Violet | `#9B7BFF` | 6.0:1 | reserved |
| Lime | `#A3E635` | 12.5:1 | reserved |
| Rose | `#FF5C8A` | 6.4:1 | reserved |
| Sky | `#4DB5FF` | 8.4:1 | reserved |

## Type

[Anton](https://fonts.google.com/specimen/Anton) for the role word of the name and section titles (uppercase only).
[Inter](https://rsms.me/inter/) Bold for overlines (0.3 em tracking) and pill labels (0.14 em); Inter Regular for
taglines and body. Both are SIL Open Font License 1.1: get them from Google Fonts
([Anton](https://fonts.google.com/specimen/Anton), [Inter](https://fonts.google.com/specimen/Inter)) or
[rsms.me/inter](https://rsms.me/inter/). In SVG assets all text is outlined to paths, so nothing depends on
installed fonts.

| Role | Banner | Social preview |
|---|---|---|
| Role word (Anton) | 100 px | 136 px |
| Overline (Inter Bold) | 17 px | 22 px |
| Tagline (Inter Regular) | 21 px | 28 px |
| Pill label (Inter Bold) | 12 px | 14 px |

## Layout rules

- **Name grammar:** `{Domain} {Role}`, for example Creator Studio. The overline is the domain, the
  large word is the role.
- **Banner:** ink background, 28 px dot grid, tile left, name block centred beside it, stage diagram right, a 5 px accent bar along the bottom edge, soft accent glow behind the end node.
- **Diagram:** stage pills, four or five (surface fill, line border) joined by `mute 2` curves to the accent end node. Fan-in puts the pills on the left, fan-out on the right.
- **Clear space:** half a tile width on every side. **Minimum size:** tile 24 px, banner 640 px wide.
- **Tagline pattern:** `Plain words in. {Result} out.` Keep it under 45 characters.
- **Avoid:** a second accent colour, gradients on the mark, third-party logos inside the mark.

## Adding a repo to the family

1. Choose a name with the `{Domain} {Role}` grammar.
2. Pick a free accent from the table (or add one that is at least 6:1 on ink) and a topology.
3. Copy `assets/brand/` from this repo, change the accent hex and, if the topology differs, the mark paths above, then rewrite the overline, role word, tagline and four stage labels in the banner and social preview.
4. Add the repo to the two tables above so each accent and topology has one owner.
5. Add the banner to the README header and use these badge settings: `style=flat-square`, `labelColor=0E1116`, colour = the repo accent.
6. Upload the social preview PNG in the repo settings.
