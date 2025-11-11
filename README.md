# Apify Web Runner — CLI + Web App

Run and inspect Apify actors from your terminal or your browser. No CDN scripts, just a lightweight React UI and a tiny CLI sharing the same core.

## What it is

Apify Web Runner helps you quickly test actors, explore their inputs, and fetch outputs:

- Web app: a local React UI to authenticate with your Apify token, load an actor, review its input schema or example input, run it, and inspect outputs (Key-Value Store record and Dataset items).
- CLI: an interactive prompt-driven tool that does the same from the terminal, with optional flags to skip prompts.

Both layers use the same shared ESM core, so behavior is consistent.

## Highlights

- One codebase, two interfaces (web + CLI)
- Reads input definitions from the actor's latest tagged build when available; otherwise falls back to example input
- Smart defaults prefilled from schema/example
- Output retrieval for KV store record (configurable key) and dataset items
- Direct output link to the KV record
- Minimal, dependency-light stack (Vite + React + apify-client)

## Quick start

Prerequisites: Node 18+.

### Web app

1) Install dependencies

```bash
npm install
```

2) Start the dev server

```bash
npm run dev
```

3) Open the printed local URL, enter your Apify API token and an actor ID (e.g., `apify/hello-world`), then run and fetch outputs.

Build and preview (optional):

```bash
npm run build
npm run preview
```

### CLI

Run interactively:

```bash
npm run cli
```

Or execute directly:

```bash
node src/cli.js
```

Pass flags to skip prompts (any input field can be provided as `--key=value`):

```bash
node src/cli.js --token=apify_api_xxx --actor=apify/hello-world --output=OUTPUT --foo=bar
```

Tip: You can also `npm link` in this repo to install a local command named after the package (currently `apify-simple`) that points to `src/cli.js`.

## How it works

The project is split into a thin UI/CLI layer and a shared core:

- `src/lib/apifyCore.js`: low-level helpers using `apify-client`
	- `ensureClient(tokenOrClient)`
	- `loadActor(tokenOrClient, actorId)`
	- `getSchemaAndDefaults(tokenOrClient, actorDetails)`
	- `runActor(tokenOrClient, actorId, input)`
	- `fetchOutputs(tokenOrClient, run, outputKey)`
- `src/lib/workflow.js`: user-flow orchestration via a tiny `uiAdapter` contract used by both CLI and the web UI
- `src/cli.js`: interactive terminal app powered by `enquire-simple`
- `src/main.jsx` + `src/App.jsx`: React UI rendered by Vite

Input handling:

1. Try to read the input schema from the actor's latest tagged build. If present, prefill defaults from schema fields (default/placeholder/prefill).
2. If no schema is found, fall back to the actor's example input JSON.

Outputs fetched:

- KV store record for a configurable key (default `OUTPUT`)
- Dataset items (listed with a sensible limit in the UI)

## Configuration and environment

- Apify authentication: supply your token in the web UI, or via `--token=…`/`APIFY_TOKEN` in the CLI
- Output key: change in the UI or pass `--output=YourKey` in CLI

## Repository layout

- `src/` — application source (web and CLI)
	- `lib/apifyCore.js` — API client helpers
	- `lib/workflow.js` — shared workflow
	- `cli.js` — CLI entry
	- `main.jsx`, `App.jsx`, `style.css` — web UI
- `vite.config.js` — build/dev config (React plugin + node polyfills)
- `package.json` — scripts and dependencies
- `dist/` — production build output

## Scripts

- `npm run dev` — start the web app in development
- `npm run build` — build the web app for production
- `npm run preview` — preview the production build locally
- `npm run cli` — run the CLI interactively

## Notes and limitations

- Some actors may not expose a formal schema; in that case the tool leverages example input when available.
- Dataset listings can be large; the UI shows a truncated view.
- Package name is `apify-simple` and the project is “Apify Web Runner.” If you `npm link`, your command name will match the package name.

## Why this exists

When iterating on actor inputs or testing store actors, it’s handy to have a fast local runner with immediate feedback and consistent behavior across terminal and browser. This tool focuses on that experience rather than deployment or CDN-based embeds.
