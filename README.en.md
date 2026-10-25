# dsh-plugin-smooth-stream

[简体中文](README.md) | English

> **This is a fork of [SpookySandwich/dsh-plugin-smooth-stream](https://github.com/SpookySandwich/dsh-plugin-smooth-stream).**
> It fixes three defects on DSH 0.2 — every message rendered twice, reasoning rows that could never reveal
> live, and a viewport that stopped following a growing reply — and adds "auto-expand the live step".
> See [What this fork changes](#what-this-fork-changes). The original work is the author's; MIT applies.

[![version](https://img.shields.io/badge/version-1.5.0-4b8dff)](https://github.com/DeerFishSheep/dsh-plugin-smooth-stream)
[![CI](https://github.com/DeerFishSheep/dsh-plugin-smooth-stream/actions/workflows/ci.yml/badge.svg)](https://github.com/DeerFishSheep/dsh-plugin-smooth-stream/actions/workflows/ci.yml)
[![license](https://img.shields.io/badge/license-MIT-blue)](LICENSE)
[![dsh](https://img.shields.io/badge/dsh-0.2.0--rc.2%20~%200.2.1--rc.3-4b8dff)](https://github.com/deepseek-ai/deepseek-harness)
[![stars](https://img.shields.io/github/stars/DeerFishSheep/dsh-plugin-smooth-stream?style=flat&label=stars)](https://github.com/DeerFishSheep/dsh-plugin-smooth-stream/stargazers)

Replaces token-by-token streaming twitch with paragraph-batched, fading reveals. Scroll follows smoothly while streaming, and reasoning blocks show a live one-line summary.

**Left: native DSH rendering · Right: Smooth Stream** — same question, started together:

![comparison demo](https://raw.githubusercontent.com/DeerFishSheep/dsh-plugin-smooth-stream/main/assets/demo.gif)

## Install

Install straight from this repository (recommended, one command):

```bash
dsh plugin --profile desktop add github:DeerFishSheep/dsh-plugin-smooth-stream
```

Without the `dsh` CLI, add it to the profile with the package manager:

```bash
cd ~/.dsh/profiles/desktop && pnpm add github:DeerFishSheep/dsh-plugin-smooth-stream
```

> Replace `desktop` with your profile name; list them with `ls ~/.dsh/profiles/`.
> GitHub access is required. This package shares its name with the npm one, so installing
> replaces it — no need to uninstall first.

Takes effect in any open session. Settings live under Settings → **Smooth Stream**. Restart or reload DSH afterwards.

## What this fork changes

On top of upstream `1.3.0`, version `1.5.0` fixes three defects and adds two settings:

| Item | Detail |
|---|---|
| Messages drawn twice | DSH 0.2 renders one assistant message twice (reasoning, then answer). This renderer drew every block in both, so each reply appeared twice. It now filters on `groupPart` exactly as the host does (upstream issue #2) |
| Reasoning never visible | A think row waited for 1000 characters before its first paragraph landed, so a short think stayed hidden for the whole step. The threshold is now 0, and text without any line break no longer stalls |
| Reasoning folded too early | Liveness used "the last array slot", so a trailing empty block or a tool call made a fresh think look superseded. It now means "the last block that carries text" |
| **Auto-expand the live step** (new) | The step being written opens itself and folds back when the next one starts. A manual click wins until that row changes role. On by default; switchable in settings |
| **Fold a think away** (new) | Choose whether a think folds when the next think starts or when the answer starts. The default keeps it open while the answer writes |
| Scroll follow | Upstream only set `scroll-behavior: smooth` and never scrolled, so a long think slid under the composer. A `ResizeObserver` now pins the port on growth, and stops as soon as the reader scrolls away |

## What it changes

- **Paragraph-batched reveal** — body text no longer jitters in token by token; it accumulates and enters a paragraph at a time with a fade. Batch boundaries avoid unclosed code fences and tables, so half-rendered Markdown never appears.
- **Smooth scroll-follow** — while streaming, the conversation glides toward the bottom covering a fraction of the remaining distance each frame: large appends catch up without teleporting. Scrolling up hands control back to you; returning to the bottom re-engages the follow.
- **Reasoning summary** — while thinking runs, the collapsed row live-scrolls its latest line, with soft fades instead of hard clipping at both edges. When a turn completes, a closing underline retracts — finished, not merely stopped.
- **Markdown & math** — body text renders through the host's own renderer: code highlighting, copy buttons, tables and KaTeX math look exactly like native output.

## Settings

The **Smooth Stream** section in the settings panel; changes apply live and persist:

| Setting | Default | Notes |
|---|---|---|
| Enable Smooth Stream | On | Off hands rendering back to the built-in view instantly — handy for comparing |
| Reveal animation | Fade | Fade / Rise / Dissolve / Wipe / Focus / Glow / Iris / Soak |
| Animation duration | 520 ms | Entrance duration (120–1200 ms) |
| Batch size | 500 chars | Text accumulated before a batch reveals; smaller feels more live, larger reads calmer |
| Smooth scroll-follow | On | The streaming-time scroll glide |
| Auto-expand live step | On | The step being written stays open; starting the next one folds it back |
| Fold a think away | When the next think starts | Either "when the next think starts", which keeps the reasoning readable through the answer, or "when the answer starts", which keeps long replies compact |

Includes a live preview and one-click reset. The settings UI follows DSH's display language.

## Reveal styles

Eight entrances, each tuned to its own rhythm; all recorded over the same reply for a direct comparison.

| Fade | Rise |
|---|---|
| ![fade](https://raw.githubusercontent.com/DeerFishSheep/dsh-plugin-smooth-stream/main/assets/variants/fade.gif) | ![rise](https://raw.githubusercontent.com/DeerFishSheep/dsh-plugin-smooth-stream/main/assets/variants/rise.gif) |
| **Dissolve** | **Wipe** |
| ![dissolve](https://raw.githubusercontent.com/DeerFishSheep/dsh-plugin-smooth-stream/main/assets/variants/dissolve.gif) | ![wipe](https://raw.githubusercontent.com/DeerFishSheep/dsh-plugin-smooth-stream/main/assets/variants/wipe.gif) |
| **Focus** | **Glow** |
| ![focus](https://raw.githubusercontent.com/DeerFishSheep/dsh-plugin-smooth-stream/main/assets/variants/focus.gif) | ![glow](https://raw.githubusercontent.com/DeerFishSheep/dsh-plugin-smooth-stream/main/assets/variants/glow.gif) |
| **Iris** | **Soak** |
| ![iris](https://raw.githubusercontent.com/DeerFishSheep/dsh-plugin-smooth-stream/main/assets/variants/iris.gif) | ![soak](https://raw.githubusercontent.com/DeerFishSheep/dsh-plugin-smooth-stream/main/assets/variants/soak.gif) |

## Compatibility

Version `1.5.0` (this repository): fixes double rendering on DSH 0.2, adds "auto-expand the live step" and "fold a think away", and makes the scroll actually follow growth. Rendering still registers on `conversation.chat.node` and `settings.section`. Verified on DSH `0.2.1-rc.3`.

Version `1.3.0` (upstream): Load on official dsh `0.2.0-rc.2`.

Version `1.2.0`: Declare the current UI renderer service provider and the verified DSH release range. Existing rendering behavior passes the current host checks.

The declared host range is `>=0.2.0-rc.2 <0.3.0-0`; official dsh `0.2.0-rc.2` satisfies it. Keep the previous plugin release on older DSH. [Validation record](.github/reviews/dsh-0.1.5.md).

To pin a version, download the `.tgz` from [GitHub Releases](https://github.com/DeerFishSheep/dsh-plugin-smooth-stream/releases) and run `dsh plugin --profile desktop add ./dsh-plugin-smooth-stream-1.5.0.tgz`; a dependency may also use `github:DeerFishSheep/dsh-plugin-smooth-stream#<commit>`.

This release targets DSH `0.2.0-rc.2`. Run `npm ci`, `npm test`, and `npm run check:package` to verify the build and package. Restart DSH after updating.


- Uses the host client services, Markdown and image APIs; automated tests cover loading and component rendering.
- This plugin owns the conversation view's assistant rendering (the `assistant-step` cell of `conversation.chat.node`) and is mutually exclusive with other plugins that take over the same rendering — with both installed, only one wins.
- Respects `prefers-reduced-motion`: with reduced motion enabled, all animation and smooth scrolling switch off.
- A rendering failure degrades to plain text for that message instead of breaking the conversation.

## License

[MIT](LICENSE)
