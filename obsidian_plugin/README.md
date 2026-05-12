# Research Frames

An Obsidian plugin that generates AI-authored research "frames" — short reframings, contrasting perspectives, and questions — over a user-selected set of notes and PDFs from an Obsidian vault. Built for the ICCC paper on computational creativity in research ideation.

This repository is the **Obsidian plugin only** (React + Chakra UI, bundled with esbuild). It talks to a separate FastAPI backend that runs on a GPU-capable host (vLLM + PostgreSQL). The backend lives in its own repository and is not shipped here — you point this plugin at a running backend URL via the plugin's settings tab.

## Prerequisites

- Node.js 16+ and npm
- An [Obsidian](https://obsidian.md) vault you can install plugins into
- A reachable Research Frames backend (URL + credentials)

## 1. Clone into your vault

The plugin must live inside an Obsidian vault's `.obsidian/plugins/` directory to load:

```bash
cd /path/to/your-vault/.obsidian/plugins
git clone <repo-url> research-frames
cd research-frames
```

## 2. Build the plugin

```bash
npm install
npm run build
```

This produces `main.js` and `main.css` next to `manifest.json`. KaTeX font files are emitted into `fonts/`.

For active development use `npm run dev` (watch mode) instead.

## 3. Enable the plugin in Obsidian

1. Open Obsidian and load the vault you cloned into.
2. Settings → Community plugins → turn off **Restricted mode** if it's on.
3. Find **Research Frames** in the installed plugins list and toggle it on.
4. Open the plugin view from the left ribbon (or the command palette: "Research Frames: Open view").

## 4. Point the plugin at a backend

Open the plugin's settings tab in Obsidian and set the backend URL (e.g. `http://your-backend-host:8000`). Without a reachable backend the plugin will not be able to authenticate or generate frames.

## 5. Use it

1. **Sign up / log in** against the backend.
2. **Setup view**: enter a research interest, then select the notes and PDFs from your vault that should seed frame generation.
3. **Generate**: pick a strategy and submit. The progress console streams LLM activity over WebSocket.
4. **Frame browser**: review, expand, search, and rank generated frames.

## Code overview

```
main.tsx                       Obsidian plugin entry — registers the view and settings tab
manifest.json                  Obsidian plugin manifest
esbuild.config.mjs             Bundler config (writes main.js, main.css, fonts/)
src/
  api.ts                       Backend HTTP + WebSocket client
  store/frameStore.ts          Zustand state (auth, frames, pagination, queue status)
  components/
    AsyncResearchView.tsx      Root view with error boundary
    views/                     LoginView, SetupView, FrameBrowserView,
                               FrameRankingView, PastRankingView, etc.
    cards/FrameCard.tsx        Frame rendering (markdown + KaTeX)
    ProgressConsole.tsx        WebSocket-driven log viewer
  theme/                       Chakra theme tuned to Obsidian's CSS variables
  utils/latexUtils.ts          LaTeX preprocessing for KaTeX
```

## License

See [LICENSE](./LICENSE).
