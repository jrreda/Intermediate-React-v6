# React Server Components (No Framework)

A minimal **Notes App** that implements React Server Components from scratch — no Next.js, no meta-framework. Just React 19, `react-server-dom-webpack`, Webpack, and a Fastify server.

Companion to [`RSCs with Next.js/`](../RSCs%20with%20Next.js/), which builds the same concepts with the Next.js App Router handling the RSC plumbing.

## Getting started

This project requires **two terminals** — one for the client bundle, one for the server.

```bash
cd RSC
npm install
```

**Terminal 1 — client bundle (run first):**

```bash
npm run dev:client
```

Webpack watches `src/Client.jsx` and writes the client bundle to `dist/`, including `react-client-manifest.json` which the server needs. Wait for the first successful compile before starting the server.

**Terminal 2 — server:**

```bash
npm run dev:server
```

Open [http://localhost:3000](http://localhost:3000).

## What you should see

- **Notes App** heading
- A notes table (server component — data from SQLite)
- A counter button (client component — interactive in the browser)

## How it works

```
Browser                    Fastify Server
   │                              │
   ├── GET / ──────────────────►  │  serves dist/index.html
   │                              │
   ├── GET /assets/*.js ───────►  │  serves webpack client bundles
   │                              │
   └── GET /react-flight ──────►  │  renderToPipeableStream(App)
                                  │  pipes RSC flight data back
```

1. **`GET /`** — Server sends `dist/index.html` (built by webpack's `HtmlWebpackPlugin`).
2. **Client boots** — `Client.jsx` fetches `/react-flight` and calls `createFromFetch` to reconstruct the React tree.
3. **`GET /react-flight`** — Server renders `App.jsx` with `renderToPipeableStream`, streaming the RSC payload. Server components run on the server; client component references are resolved via the webpack manifest.
4. **Hydration** — Client components (marked `"use client"`) hydrate in the browser with their JS bundles.

## Components

| File | Type | Role |
|---|---|---|
| `src/App.jsx` | Server | Root component — composes server + client children inside `<Suspense>` |
| `src/ServerComponent.jsx` | Server | Async component — queries `notes.db` for user 1's sent notes |
| `src/ClientComponent.jsx` | Client (`"use client"`) | Interactive counter with `useState` |
| `src/Client.jsx` | Client entry | Fetches the flight stream and mounts the app with `createRoot` |

Server component logs appear in the **server terminal**. Client component logs appear in the **browser console**.

## Project structure

```
src/
├── App.jsx              Root server component
├── ServerComponent.jsx  Async server component (SQLite)
├── ClientComponent.jsx  Client component (counter)
└── Client.jsx           Client entry point

server/
├── main.js              Registers RSC + Babel, starts server
└── server.js            Fastify routes (/ and /react-flight)

dist/                    Webpack output (client bundles + manifest)
notes.db                 SQLite database (users + notes)
index.html               HTML template for webpack
webpack.config.js        Client bundle + ReactServerWebpackPlugin
babel.config.js          JSX transform for client components
```

## Key pieces

### `react-server-dom-webpack`

The bridge between server and client without a framework:

- **`ReactServerWebpackPlugin`** (webpack) — generates `react-client-manifest.json`, mapping client component modules to their JS chunks
- **`renderToPipeableStream`** (server) — serializes the server component tree into a flight stream
- **`createFromFetch`** (client) — deserializes the flight stream back into a React tree
- **`node-register`** (server) — enables `"use client"` directive handling on the server

### Server setup (`server/main.js`)

Registers two transforms before loading the server:

1. `react-server-dom-webpack/node-register` — RSC module resolution
2. `@babel/register` — transpiles `src/*.jsx` on the fly (CommonJS for Node)

### Client manifest

The server reads `dist/react-client-manifest.json` at startup. If you start the server before webpack has built, it will crash. Always run `dev:client` first.

## Database

`notes.db` in the project root contains **users** and **notes** tables. `ServerComponent.jsx` queries notes sent by user 1:

```sql
SELECT ... FROM notes WHERE from_user = 1
```

The database path is resolved relative to `src/`:

```js
path.resolve(__dirname, "../notes.db")
```

## Scripts

| Command | Description |
|---|---|
| `npm run dev:client` | Webpack watch — rebuilds client bundle on change |
| `npm run dev:server` | Node watch — restarts server on change (`--conditions react-server`) |

There is no single `npm run dev` — both processes must run simultaneously.

## Troubleshooting

**Blank page**

Check the server terminal for errors on `/react-flight`. A common cause is a bad SQLite path or missing tables — server component errors break the flight stream and nothing renders in the browser.

**`"root" path ".../public" must exist` warning**

The server serves static files from a `public/` folder (for `/index.css`). Create it and copy `index.css` there if you want styles to load:

```bash
mkdir public && cp index.css public/
```

**Client changes not reflected**

Webpack rebuilds automatically. Refresh the browser.

**Server component changes not reflected**

The server restarts via `--watch`. Refresh the browser to fetch a new flight stream.

## Tech stack

- React 19 + `react-server-dom-webpack`
- Webpack 5 + Babel
- Fastify + `@fastify/static`
- SQLite via `promised-sqlite3`
- [doodle.css](https://github.com/jaywcjlove/doodle.css)

## vs. Next.js

| | This project | `RSCs with Next.js/` |
|---|---|---|
| Flight stream | Manual `renderToPipeableStream` + Fastify | Handled by Next.js |
| Client manifest | Webpack + `ReactServerWebpackPlugin` | Built into Next.js |
| Dev workflow | Two terminals (client + server) | Single `npm run dev` |
| Server Actions | Not implemented | `"use server"` functions |
| Routing | Single page | App Router file-based routes |
| `"use client"` | Webpack plugin + manifest | Framework convention |
