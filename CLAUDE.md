# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Is

ETHSKILLS is a content-driven knowledge base that corrects common AI/LLM misconceptions about Ethereum development. Each "skill" is a standalone markdown file served as a static URL that AI agents can fetch to update their context. Deployed at https://ethskills.com via Vercel.

There is no build step, no package manager, no dependencies, and no tests. This is a static site — just HTML, CSS, JS, and markdown files.

## Repository Structure

- `index.html` — Single-page static site (amber CRT aesthetic, vanilla JS, modal-based skill reader)
- `SKILL.md` — Combined file aggregating all skills (~133KB), regenerated after merges
- `<topic>/SKILL.md` — Individual skill files (gas, wallets, l2s, standards, tools, etc.)
- Redirect stubs: `contracts/`, `defi/`, `l2/`, `layer2/` redirect to their renamed counterparts
- `.github/workflows/update-gas.yml` — Weekly GitHub Action that fetches live gas prices and ETH price, updates `gas/SKILL.md` freshness stamp

## Skill File Format

Every `SKILL.md` starts with YAML frontmatter:

```
---
name: topic-name
description: One-line description for bots to parse.
---
```

Content follows a consistent pattern: "What You Probably Got Wrong" section first, then detailed corrections with code examples, tables, and verified data.

## Key Conventions

- **"onchain" not "on-chain"** — One word, no hyphen. This matches Ethereum community convention.
- **Contract addresses must be verified** — Never fabricate addresses. All addresses in `addresses/SKILL.md` are checked against block explorers using `cast code` and `cast call`.
- **Content must counter actual LLM mistakes** — If LLMs already know something correctly, don't include it. Test claims against current onchain reality.
- **Redirect stubs for moved content** — When a skill is renamed, leave a stub file pointing to the new URL for backward compatibility.

## Working With Content

- Edit individual skill files in their directories (e.g., `gas/SKILL.md`)
- The root `SKILL.md` is a generated aggregate — update individual skills, then regenerate
- Open `index.html` in a browser to test locally (skills load via relative paths)
- Push to `master` to deploy via Vercel

## Automation

The only CI is `.github/workflows/update-gas.yml`: runs Mondays at noon UTC, fetches gas/ETH prices from llamarpc.com and CoinGecko, updates the freshness line in `gas/SKILL.md`, and auto-commits if changed.
