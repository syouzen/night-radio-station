# Night Radio Station

Story-driven idle desktop game about running a midnight radio station for a sleepless city.

## Quick Start

```bash
npm install
npm run dev
```

For the desktop app:

```bash
npm run tauri dev
```

Tauri desktop development requires the Rust toolchain.

## Commands

| Command | Description |
| --- | --- |
| `npm run dev` | Start the SvelteKit dev server. |
| `npm run tauri dev` | Start the Tauri desktop app. |
| `npm run check` | Run Svelte type checking. |
| `npm run build` | Build the static frontend. |
| `npm run preview` | Preview the production frontend build. |

## Current MVP Loop

- Broadcast stays on air while the app is open.
- Listener letters arrive over time.
- Signal, listeners, reputation, and story fragments grow through idle play.
- Antenna and transmitter upgrades unlock stronger station progress.
- Progress saves locally in browser storage for the first slice.
