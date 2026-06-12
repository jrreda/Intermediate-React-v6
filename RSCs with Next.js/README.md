# React Server Components with Next.js

A **Note Passer** demo app built with the Next.js App Router, showing how React Server Components, Server Actions, and client components work together — with the framework handling the RSC wiring for you.

Companion to the hand-rolled [`RSC/`](../RSC/) example in this repo, which implements the same ideas without a framework.

## Getting started

```bash
cd "RSCs with Next.js"
npm install
npm run dev
```

Open [http://localhost:3000](http://localhost:3000). The dev server uses Turbopack (`next dev --turbopack`).

## App overview

**Note Passer** is a small notes app backed by SQLite (`notes.db`). Users can read notes, write new ones, and a "teacher" view polls for new notes in real time.

The app assumes you are logged in as **user 1** (no real auth — just hard-coded queries).

## Routes

| Route | What it demonstrates |
|---|---|
| `/` | Home page with navigation links |
| `/my` | **Server Component** — async data fetch from SQLite, rendered on the server |
| `/write` | **Server Component** + **Server Action** — form rendered on the server, submitted via `postNote` |
| `/teacher` | **Server → Client boundary** — server fetches initial notes, client polls for updates via a Server Action |
| `/who-am-i` | **Server/Client composition** — server component nested inside a client component, with a Server Action form |

## RSC patterns in this project

### Server Components (default)

Every file in `src/app/` is a Server Component unless marked with `"use client"`. These run only on the server and can access the database directly:

- `src/app/my/page.js` — queries notes sent to and from user 1
- `src/app/write/page.js` — loads users for the form dropdowns
- `src/app/who-am-i/whoAmiI.js` — reads the current user from SQLite

### Server Actions (`"use server"`)

Functions that run on the server but can be called from forms or client components:

- `src/app/write/postNote.js` — inserts a new note from the write form
- `src/app/teacher/fetchNotes.js` — fetches notes (initial load + polling)
- `src/app/who-am-i/updateUsername.js` — updates a username and redirects home

### Client Components (`"use client"`)

Interactive UI that runs in the browser:

- `src/app/teacher/clientPage.js` — polls `fetchNotes` every 5 seconds for new notes
- `src/app/who-am-i/clientPage.js` — wraps the server `WhoAmI` component and renders the username form

### Server → Client data flow

The teacher page is the clearest example of passing server data into client code:

1. `teacher/page.js` (server) calls `fetchNotes()` and passes the result to the client component
2. `teacher/clientPage.js` (client) receives `initialNotes` and the `fetchNotes` Server Action as props
3. The client polls for new notes on an interval without a separate API route

## Project structure

```
src/app/
├── layout.js              Root layout (nav, doodle.css)
├── page.js                Home
├── my/page.js             My Notes (server component)
├── write/
│   ├── page.js            Write form (server component)
│   └── postNote.js        Server Action
├── teacher/
│   ├── page.js            Teacher view (server → client)
│   ├── clientPage.js      Client component (polling)
│   └── fetchNotes.js      Server Action
└── who-am-i/
    ├── page.js            Who Am I page (server + client)
    ├── whoAmiI.js         Server component
    ├── clientPage.js      Client component (form)
    └── updateUsername.js  Server Action

notes.db                   SQLite database (users + notes)
```

## Database

The app uses `notes.db` in the project root with two tables:

- **users** — id, name
- **notes** — id, from_user, to_user, note

All database access uses [`promised-sqlite3`](https://www.npmjs.com/package/promised-sqlite3) with paths relative to the project root (`./notes.db`).

## Scripts

| Command | Description |
|---|---|
| `npm run dev` | Start dev server with Turbopack |
| `npm run build` | Production build |
| `npm run start` | Serve production build |
| `npm run lint` | Run ESLint |

## Tech stack

- [Next.js 15](https://nextjs.org/) (App Router, Turbopack)
- React 19
- SQLite via `promised-sqlite3`
- [doodle.css](https://github.com/jaywcjlove/doodle.css) for styling

## vs. hand-rolled RSC

| | `RSC/` (no framework) | This project |
|---|---|---|
| Flight stream | Manual `renderToPipeableStream` + Fastify | Handled by Next.js |
| Client manifest | Webpack + `react-server-dom-webpack` plugin | Built into Next.js |
| Server Actions | Not implemented | `"use server"` functions |
| Routing | Single page | App Router file-based routes |
| Data fetching | Inline in server components | Same pattern, framework-managed |
