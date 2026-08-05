# IT Helper

A step-by-step troubleshooting chat assistant for non-technical school staff,
built for a small education nonprofit's help desk. It walks users through
common IT issues (passwords, printers, Wi-Fi, Teams, projectors, etc.) before
they submit a ticket, and can generate a formatted ticket summary when an
issue can't be self-resolved.

## Getting started

```bash
npm install
cp .env.example .env   # then fill in VITE_ANTHROPIC_API_KEY
npm run dev
```

Open the URL Vite prints (defaults to http://localhost:5173).

## Project structure

```
src/
  components/ITHelper.jsx   # the chat UI + handbook system prompt
  App.jsx
  main.jsx
docs/
  handbook.md                # source handbook content used to write the system prompt
```

## Important: API key exposure

This app calls the Anthropic API directly from the browser using
`VITE_ANTHROPIC_API_KEY`. That's fine for local development or an internal
tool behind a firewall, but it exposes the key to anyone who loads the page.
**Do not deploy this publicly as-is.** For a real deployment, put a small
backend (or serverless function) in front of the Anthropic API that holds the
key server-side, and have the frontend call that instead.

## Scripts

- `npm run dev` — start the dev server
- `npm run build` — production build
- `npm run preview` — preview the production build
- `npm run lint` — lint with oxlint
