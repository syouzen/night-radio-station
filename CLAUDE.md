# Night Radio Station

Story-driven idle desktop game built with Tauri 2, SvelteKit, and TypeScript.

## Product Direction

- Player runs a midnight radio station in a small always-on desktop window.
- Idle progression should unlock listener letters, strange frequencies, city stories, and emotional narrative beats.
- Prefer atmosphere, readable state, and small meaningful choices over complex combat or mobile-style monetization.

## Commands

| Command | Purpose |
| --- | --- |
| `npm install` | Install frontend and Tauri CLI dependencies. |
| `npm run dev` | Run SvelteKit dev server only. |
| `npm run tauri dev` | Run desktop app. Requires Rust toolchain. |
| `npm run check` | Run Svelte type checking. |
| `npm run build` | Build static frontend output for Tauri. |

## Working Rules

- Keep MVP vertical: one playable loop before adding systems.
- Use local-first persistence unless a feature clearly needs network support.
- Keep UI accessible: semantic buttons, visible focus, readable contrast, responsive layouts.
- Do not introduce server, account system, payment, cloud save, or multiplayer without explicit product reason.
- Run `npm run check` and `npm run build` after frontend changes.
- For Tauri runtime verification, run `npm run tauri dev` when Rust is installed.
