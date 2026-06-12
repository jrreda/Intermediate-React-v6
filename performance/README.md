# React Performance

A markdown editor demo that highlights unnecessary re-renders and how to avoid them with `memo`, `useMemo`, and stable props.

## Getting started

```bash
cd performance
npm install
npm run dev
```

Open the URL Vite prints (usually [http://localhost:5173](http://localhost:5173)).

## What you should see

- A **clock** that updates every second
- A **theme** dropdown (green, blue, red, yellow)
- A **markdown editor** (textarea) with sample content
- A **preview** pane showing rendered HTML

Watch the **Last Render** timestamp in the preview. It should only change when you edit the text or switch themes — not every second when the clock ticks.

## The problem

`App` updates `time` every second with `setInterval`. Without optimizations, that state change re-renders the entire tree, including `MarkdownPreview`.

The preview does two expensive things on every render:

1. **Parses markdown** with `marked.parse()`
2. **Simulates jank** — a 100ms busy loop in `MarkdownPreview` to make slow renders obvious

Re-rendering the preview once per second would make typing feel sluggish, even though the markdown and theme did not change.

## The solution

### `memo` on the child

`MarkdownPreview` is wrapped in `React.memo` so it skips re-renders when props are unchanged:

```jsx
export default memo(function MarkdownPreview({ render, options }) { ... });
```

### Stable props with `useMemo`

`App` memoizes the props passed to the preview so they keep the same reference between clock ticks:

```jsx
const options = useMemo(() => ({ text, theme }), [text, theme]);
const render = useMemo(() => (text) => marked.parse(text), []);
```

- `**options**` — new object only when `text` or `theme` changes
- `**render**` — stable parse function (empty dependency array)

A commented `useCallback` alternative exists in `App.jsx` for the same `render` function:

```jsx
// const render = useCallback((text) => marked.parse(text), []);
```

## How to experiment

1. **Confirm memo works** — the clock ticks, but "Last Render" stays fixed until you edit text or change theme.
2. **Remove `memo`** from `MarkdownPreview.jsx` — "Last Render" updates every second; typing may feel laggy.
3. **Remove `useMemo`** for `options` or `render` — `memo` stops helping because props get new references every render.
4. **Toggle `useMemo` vs `useCallback`** for `render` — both keep the function reference stable.

Use React DevTools **Profiler** or watch "Last Render" to see when the preview actually re-renders.

## Project structure

```
src/
├── App.jsx              Main app — clock, theme, editor, memoized props
├── MarkdownPreview.jsx  Memoized preview with artificial 100ms jank
├── markdownContent.js   Default markdown sample text
├── Client.jsx           Client entry point
└── style.css            Layout and theme styles

index.html               HTML shell
vite.config.js           Vite + React plugin
```

## Request flow

```
App (re-renders every second)
  │
  ├── time state updates → h2 re-renders
  ├── theme / text unchanged
  │
  └── MarkdownPreview (memo)
        └── props unchanged → skip re-render ✓
```

When you type or change theme:

```
App
  └── options reference changes
        └── MarkdownPreview re-renders → marked.parse() + 100ms jank
```

## Scripts


| Command              | Description             |
| -------------------- | ----------------------- |
| `npm run dev`        | Start Vite dev server   |
| `npm run dev:client` | Alias for `npm run dev` |


## Key APIs


| API           | Role in this demo                                                       |
| ------------- | ----------------------------------------------------------------------- |
| `memo`        | Skip child re-render when props are shallow-equal                       |
| `useMemo`     | Stable `options` object and `render` function between parent re-renders |
| `useCallback` | Alternative for stabilizing the `render` function                       |
| `useEffect`   | Interval that drives the clock (intentional parent re-renders)          |


## Tech stack

- React 19
- [Vite](https://vite.dev/) + [@vitejs/plugin-react](https://github.com/vitejs/vite-plugin-react)
- [marked](https://marked.js.org/) (markdown parsing)

