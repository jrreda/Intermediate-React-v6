# Static Site Generation (SSG)

The simplest React rendering mode in this repo — render components to HTML **once at build time** and ship a plain static file. No server, no client JavaScript, no hydration.

## Getting started

```bash
cd SSG
npm install
npm run build
```

Open `dist/index.html` in a browser, or serve the `dist/` folder:

```bash
npx serve dist
```

You should see:

> **Hello Frontend Masters**
> This is SSG

## How it works

```
index.html          build.js              dist/index.html
┌─────────────┐     ┌──────────────┐      ┌─────────────────────────┐
│ <!--ROOT--> │ ──► │ renderTo     │ ──►  │ <div><h1>Hello...</h1>  │
│  placeholder│     │ StaticMarkup │      │ <p>This is SSG</p></div>│
└─────────────┘     └──────────────┘      └─────────────────────────┘
                           ▲
                      App.js (React)
```

1. `**build.js**` reads `index.html` as an HTML shell.
2. `**renderToStaticMarkup**` converts the React component tree in `App.js` to an HTML string.
3. The `<!--ROOT-->` placeholder is replaced with that HTML.
4. The final file is written to `dist/index.html`.

There is no dev server and no watch mode — run `npm run build` again after making changes.

## Project structure

```
App.js          React component (uses createElement, no JSX)
index.html      HTML shell with <!--ROOT--> placeholder
build.js        Build script — renders React to static HTML
dist/           Build output (generated)
```

## Key details

### `renderToStaticMarkup`

Unlike `renderToString` (used in SSR), `renderToStaticMarkup` produces HTML without React-specific attributes (`data-reactroot`, etc.). The output is meant to be served as-is — not hydrated.

### No JSX, no bundler

`App.js` uses `createElement` directly so Node can import it without Babel or a bundler:

```js
import { createElement as h } from "react";

function App() {
  return h("div", null,
    h("h1", null, "Hello Frontend Masters"),
    h("p", null, "This is SSG"),
  );
}
```

### Build output

Before build, `index.html` contains a placeholder:

```html
<div id="root"><!--ROOT--></div>
```

After build, `dist/index.html` contains the rendered markup:

```html
<div id="root"><div><h1>Hello Frontend Masters</h1><p>This is SSG</p></div></div>
```

The `dist/` folder is cleaned on each build — old files are removed before writing the new `index.html`.

## Scripts


| Command         | Description                                      |
| --------------- | ------------------------------------------------ |
| `npm run build` | Render React to HTML and write `dist/index.html` |


## When to use SSG

Static generation works well when:

- Content is the same for every visitor
- Pages can be pre-built ahead of time
- No interactivity is needed in the browser

This demo has no buttons, forms, or state — everything is baked into HTML at build time.

## vs. SSR in this repo


|                        | SSG (this project)     | `[SSR/](../SSR/)`                |
| ---------------------- | ---------------------- | -------------------------------- |
| When HTML is generated | Build time             | Every request                    |
| Server required        | No                     | Yes (Fastify)                    |
| Client JavaScript      | None                   | Yes (Vite bundle)                |
| Interactivity          | No                     | Yes (hydration)                  |
| React API              | `renderToStaticMarkup` | `renderToString` + `hydrateRoot` |
| Dev experience         | Build, then open file  | Dev server on port 3000          |


## Tech stack

- React 19
- Node.js (ES modules)
- No framework, no bundler, no transpiler

