# React Rendering Modes

Minimal, from-scratch examples of React rendering strategies for [Frontend Masters — Intermediate React v6](https://frontendmasters.com/courses/intermediate-react-v6/).

Each folder is a self-contained demo with no framework abstractions — just React, Node, and a small build step.

## Project structure

```
├── SSG/   Static Site Generation — HTML generated at build time
└── SSR/   Server-Side Rendering — HTML generated per request, then hydrated on the client
```

## SSG — Static Site Generation

React is rendered to HTML once at build time. The output is a plain static file with no client-side JavaScript.

**How it works**

1. `build.js` reads `index.html` and finds the `<!--ROOT-->` placeholder.
2. `renderToStaticMarkup` turns the React tree into HTML.
3. The result is written to `dist/index.html`.

**Run it**

```bash
cd SSG
npm install
npm run build
```

Open `SSG/dist/index.html` in a browser, or serve the `dist` folder with any static file server.

## SSR — Server-Side Rendering

React is rendered to HTML on every request. The client bundle hydrates the page so interactive components (like the counter button) work in the browser.

**How it works**

1. `npm run build` uses Vite to bundle `client.js` into `dist/`.
2. `server.js` (Fastify) reads the built HTML shell and splits it at `<!--ROOT-->`.
3. On each `GET /`, `renderToString` renders the React app server-side and streams the response.
4. `client.js` calls `hydrateRoot` to attach event handlers on the client.

**Run it**

```bash
cd SSR
npm install
npm run build
npm start
```

Visit [http://localhost:3000](http://localhost:3000). The counter button should be interactive after hydration.

## Comparison

| | SSG | SSR |
|---|---|---|
| When HTML is generated | Build time | Request time |
| Server required | No (static files) | Yes |
| Client JavaScript | None | Yes (hydration) |
| Interactivity | No | Yes |
| React API | `renderToStaticMarkup` | `renderToString` + `hydrateRoot` |

## Tech stack

- React 19
- Node.js (ES modules)
- [Fastify](https://fastify.dev/) + [@fastify/static](https://github.com/fastify/fastify-static) (SSR server)
- [Vite](https://vite.dev/) (SSR client bundle)
