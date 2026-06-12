# Server-Side Rendering (SSR)

React rendered to HTML **on every request**, then **hydrated** in the browser so interactive components work. A minimal from-scratch SSR setup with Fastify and Vite.

## Getting started

```bash
cd SSR
npm install
npm run build
npm start
```

Visit [http://localhost:3000](http://localhost:3000).

You should see:

> **Hello Frontend Masters**
> This is SSR

The **Count** button should be interactive — click it to increment the counter.

## How it works

```
Browser                         Fastify Server
   │                                  │
   ├── GET / ─────────────────────►   │  1. Read dist/index.html shell
   │                                  │  2. Split at <!--ROOT-->
   │                                  │  3. renderToString(App)
   │  ◄── HTML + React markup ──────  │  4. Stream: shell + HTML + shell
   │                                  │
   ├── GET /assets/*.js ───────────►  │  Serve Vite client bundle
   │                                  │
   └── hydrateRoot(App) ────────────  │  Client attaches event handlers
```

### Build step (`npm run build`)

Vite bundles `client.js` (and its dependencies) into `dist/`. It also processes `index.html`, replacing the `./client.js` script tag with the hashed bundle path:

```html
<script type="module" src="/assets/index-*.js"></script>
```

The `<!--ROOT-->` placeholder stays in `dist/index.html` for the server to fill in at request time.

### Request handling (`server.js`)

On each `GET /`:

1. The pre-read HTML shell is split at `<!--ROOT-->`.
2. `renderToString` converts `App.js` to an HTML string on the server.
3. The response is streamed: opening shell + React markup + closing shell.

Static assets in `dist/` (JS bundles) are served by `@fastify/static`.

### Hydration (`client.js`)

After the HTML loads, the client bundle calls `hydrateRoot` to attach React event handlers to the server-rendered DOM. Without this step, the counter button would render but not respond to clicks.

## Project structure

```
App.js          Shared React component (server + client)
client.js       Client entry — hydrates the app
index.html      HTML shell with <!--ROOT--> placeholder
server.js       Fastify server — renderToString per request
dist/           Vite build output (client bundle + processed HTML)
```

## Key details

### Shared component

`App.js` is imported by both the server and the client:

- `**server.js**` — imports directly in Node, renders with `renderToString`
- `**client.js**` — bundled by Vite, hydrates with `hydrateRoot`

The component uses `createElement` (no JSX) so Node can import it without a transpiler:

```js
import { createElement as h, useState } from "react";

function App() {
  const [count, setCount] = useState(0);
  return h("div", null,
    h("h1", null, "Hello Frontend Masters"),
    h("p", null, "This is SSR"),
    h("button", { onClick: () => setCount(count + 1) }, `Count: ${count}`),
  );
}
```

### `renderToString` vs `renderToStaticMarkup`

SSR uses `renderToString`, which includes extra attributes React needs for hydration. SSG uses `renderToStaticMarkup`, which strips those attributes because there is no client-side React.

### Streaming response

The server writes the response in three parts rather than building one big string:

```js
reply.raw.write(parts[0]);              // <html>...<div id="root">
reply.raw.write(renderToString(h(App))); // rendered React HTML
reply.raw.write(parts[1]);              // </div>...</html>
```

This pattern mirrors how production SSR servers stream HTML to the browser.

## Scripts


| Command         | Description                          |
| --------------- | ------------------------------------ |
| `npm run build` | Vite bundles the client into `dist/` |
| `npm start`     | Start Fastify server on port 3000    |


Run `npm run build` before `npm start`, and again after changing client-side code. Server-only changes to `App.js` or `server.js` only require restarting the server.

## Troubleshooting

**Counter button doesn't work**

The client bundle may be stale or missing. Run `npm run build` and refresh the page. Check the browser Network tab — the `/assets/index-*.js` file should return 200.

**Blank or broken page**

Make sure `dist/index.html` exists (run `npm run build` first). The server reads this file at startup.

**Port already in use**

Another process (or the `RSC/` dev server) may be on port 3000. Stop it or set `PORT=3001 npm start`.

## vs. SSG in this repo


|                        | `[SSG/](../SSG/)`      | SSR (this project)               |
| ---------------------- | ---------------------- | -------------------------------- |
| When HTML is generated | Build time             | Every request                    |
| Server required        | No                     | Yes (Fastify)                    |
| Client JavaScript      | None                   | Yes (Vite bundle)                |
| Interactivity          | No                     | Yes (hydration)                  |
| React API              | `renderToStaticMarkup` | `renderToString` + `hydrateRoot` |
| Build tool             | Node script            | Vite                             |


## Tech stack

- React 19
- Node.js (ES modules)
- [Fastify](https://fastify.dev/) + [@fastify/static](https://github.com/fastify/fastify-static)
- [Vite](https://vite.dev/) (client bundle)

