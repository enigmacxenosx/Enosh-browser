# Enosx Browser

A private, focused, offline-ready browser shell with Enosx AI.

## What is implemented

Enosx Browser now includes a Chrome/Brave-style browser shell with tabs, address/search navigation, back and forward controls, reload, bookmarks, reading list, history, workspaces, private mode, privacy status, install prompts, and an Enosx AI panel.

Enosx AI includes explicit context permissions and selectable processing modes: Local AI, Hybrid AI, Private Cloud AI, and No AI. The Enosx AI sidebar now includes a **Local GGUF models** manager: import a `.gguf` file downloaded from Hugging Face, inspect its GGUF header and quantization, choose it as the active local model, and remove it later. The model blob and metadata are stored in IndexedDB on the device and are not uploaded.

### Offline GGUF workflow

1. Open the Enosx AI sidebar with the brain icon.
2. Under **Local GGUF models**, choose **Import** and select a Hugging Face `.gguf` file.
3. Select the imported model, keep **Processing mode** on **Local AI**, and submit a prompt.

The **Generation controls** card lets you tune the next local run without leaving the sidebar:

- **Temperature** — lower values are more focused; higher values are more creative.
- **Context window** — choose how much conversation/page context the runtime can consider.
- **Max output** — cap the number of generated tokens.
- **Top P** — adjust nucleus sampling for another creativity/focus tradeoff.

These preferences are saved in local storage and are included in the prompt handoff contract for the future native inference runtime.

### Local AI run states

The sidebar now communicates the full local-run lifecycle:

- **Loading local model** — prepares the selected GGUF and context window.
- **Generating response** — shows the active prompt, animated token indicator, and a **Stop** action.
- **Generation unavailable** — explains when the native inference runtime is not connected and offers **Retry** without losing the prompt.

The UI does not display a fake completion. Until a native runtime is attached, the demo transitions to the unavailable state while keeping the model and prompt local.

The browser shell validates and stores the model locally so it remains available offline. This UI is deliberately runtime-agnostic: the next native release can attach a bundled `llama.cpp`/`llama-server` runtime without changing the sidebar or model library. Until that runtime is bundled, the panel clearly reports that the selected model is ready and keeps the prompt local instead of pretending a remote completion occurred.

The browser is offline-first. The app registers a service worker, caches the application shell, displays online/offline status, and stores bookmarks, reading-list items, history, and preferences in local browser storage. The installed desktop app loads the compiled static build without requiring an internet connection.

## Install like a desktop browser

### Linux installer artifacts

The repository includes an Electron Builder configuration. A Linux build produces:

- `release/Enosx-Browser-0.1.0-linux-x86_64.AppImage` — portable installer-free application. Make it executable with `chmod +x` and launch it directly.
- `release/Enosx-Browser-0.1.0-linux-amd64.deb` — Debian/Ubuntu installer. Install with `sudo apt install ./release/Enosx-Browser-0.1.0-linux-amd64.deb`.

These files are generated locally and are intentionally excluded from Git so that release artifacts can be uploaded separately or attached to a GitHub Release.

### Windows and macOS installers

Build the installers on the target operating system:

```bash
pnpm installer:windows   # NSIS .exe installer
pnpm installer:mac       # .dmg installer
```

Electron Builder packages the same static offline build into a native desktop application. Cross-platform builds may require platform-specific signing and packaging dependencies.

### Browser installation from Chrome or Brave

The web build is also installable as a Progressive Web App. Start or deploy the production web build, open it in Chrome or Brave, then choose **Install Enosx Browser** from the address-bar install control or the browser menu. The PWA uses `public/manifest.webmanifest` and `public/sw.js` for installation and offline app-shell caching.

## Development

```bash
pnpm install
pnpm dev
```

Open `http://localhost:3000` in a browser. The desktop development shell can be started with:

```bash
pnpm electron:dev
```

## Production builds

```bash
pnpm build:web
pnpm electron:build
```

`pnpm build:web` creates the static `out/` directory. `pnpm electron:build` packages it as a desktop application using the configuration in `package.json` and `electron/`.

## Offline behavior

The first web visit must be online so the browser can download the app shell. After that, the interface, local navigation shell, saved bookmarks, reading list, history, privacy controls, imported GGUF model files, and local AI model-selection flow remain available offline. External websites, remote search, cloud AI, and external synchronization still require a network connection.

## Security notes

The Electron shell uses context isolation, disables Node integration in the renderer, enables sandboxing, and restricts external window requests to the system browser. GGUF files are kept in the renderer’s IndexedDB storage and are only accepted after the file magic is validated. The current release provides the offline model library and UI contract; a native inference runtime is still required for token generation.
