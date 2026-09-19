# Base44 Dev Environment

## Project Overview

This is a single-file React quiz game ("Líderes & Equipos: El Reto CNP") that was
provided as a raw component file (`pasted.txt` / `Mini-Juego`). A Vite + React +
Tailwind CSS project was scaffolded around it so it can run in development.

## Tech Stack

- **Vite 5** + **React 18** (dev server on port 5173, mapped to host port 3000)
- **Tailwind CSS 3** (PostCSS plugin)
- **lucide-react** for icons

## Running the App

```bash
docker compose -f docker-compose.base44.yml up -d --build
```

The dev server auto-installs npm dependencies on first start, then runs
`vite --host 0.0.0.0 --port 5173` with live reload enabled.

## Key Files

- `src/App.jsx` — the entire quiz game component (copied from `pasted.txt`)
- `src/main.jsx` — React entry point
- `src/index.css` — Tailwind directives
- `vite.config.js` — Vite config (React plugin, binds 0.0.0.0)
- `tailwind.config.js` / `postcss.config.js` — Tailwind setup

## Notes

- The original `Mini-Juego` file has a typo on line 1 (`dimport`); `pasted.txt`
  is the clean version and was used as `src/App.jsx`.
- No external secrets or database required — purely client-side.
