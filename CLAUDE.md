# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

ReactFlux is a third-party web frontend for Miniflux (RSS reader) built with React and Vite. It's a single-page application (SPA) that generates static files for deployment to platforms like Cloudflare Pages, Vercel, or self-hosted environments.

## Development Commands

### Core Development
- `pnpm dev` - Start development server on `http://0.0.0.0:3000`
- `pnpm build` - Build production bundle to `build/` directory
- `pnpm preview` - Build and preview production bundle locally

### Code Quality
- `pnpm lint` - Run ESLint to check code
- `pnpm lint:fix` - Run ESLint and auto-fix issues
- `pnpm format` - Format all files with Prettier
- `pnpm format:check` - Check formatting without modifying files

### Other
- `pnpm update-fonts` - Update font files using the update-fonts.js script
- Pre-commit hooks automatically run `lint-staged` (ESLint + Prettier on staged files)

## Architecture Overview

### State Management
The app uses **Nanostores** (not Redux or Context API) for state management with persistent atoms:
- `authState` - Authentication (server URL, token/username/password)
- `settingsState` - User preferences (theme, layout, filters, reading options)
- `contentState` - Current view state (active article, feed, filters)
- `dataState` - Cached feed/article data
- `feedIconsState` - Feed icon cache
- `hotkeysState` - Keyboard shortcut configuration
- `sidebarState` - Sidebar UI state

Access stores with `useStore()` hook from `@nanostores/react` or direct `.get()/.set()` calls.

### Routing
React Router v7 with dynamic route generation in `src/routes.jsx`:
- Routes are programmatically created from `pageRoutes` object
- Each route has both list view (`/all`) and detail view (`/all/entry/:entryId`) patterns
- Protected by `RouterProtect` component (checks authentication)
- Home page determined by `homePage` setting

### API Layer
Located in `src/apis/`:
- `ofetch.js` - Configured HTTP client with auth headers, retry logic, 401 handling
- API modules: `categories.js`, `entries.js`, `feeds.js`
- All requests auto-include Miniflux server URL and auth headers from `authState`

### Component Structure
- `src/components/` - Organized by feature (Article, Content, Main, Sidebar, Settings, ui)
- Components are in PascalCase directories with PascalCase `.jsx` files
- Hooks in `src/hooks/` use camelCase naming (e.g., `useAppData.js`)
- Store files in `src/store/` use camelCase naming (e.g., `settingsState.js`)

### Custom Hooks Pattern
Extensive use of custom hooks for logic separation:
- `useAppData` - Fetch and manage feeds/categories/entries
- `useArticleList` - Article list filtering/pagination logic
- `useKeyHandlers` - Keyboard shortcut handling
- `useEntryActions` - Article actions (mark read, star, save)
- `useFeedOperations` - Feed management operations
- `useTheme` - Theme switching and CSS custom properties
- `useLanguage` - i18n with Polyglot library

### Internationalization
- Locales in `src/locales/` (de-DE, en-US, es-ES, fr-FR, zh-CN)
- Uses `node-polyglot` for translations
- Arco Design components also need locale imports (see `App.jsx`)
- Language detection via `getBrowserLanguage()` utility

## Code Style

### ESLint Rules (eslint.config.mjs)
- Uses flat config format
- Enforces import ordering (alphabetical, grouped by type)
- No relative parent imports (`import/no-relative-parent-imports`)
- React Compiler plugin enabled
- Sorted JSX props (callbacks last, reserved first, shorthand first)
- Filename conventions:
  - `.jsx` components: PascalCase
  - `src/hooks/`: camelCase
  - `src/store/`: camelCase

### Prettier Configuration
- No semicolons
- Double quotes (not single)
- 100 character line width
- Trailing commas everywhere
- 2 space indentation

### Import Style
Imports must follow strict ordering (enforced by ESLint):
1. Node built-ins
2. External packages
3. Internal modules (using `@/` alias)
4. Parent/sibling imports
5. Type imports

Always use `@/` alias for src imports (configured in `vite.config.js`).

## Build Configuration

### Vite Setup (vite.config.js)
- React Compiler enabled via Babel plugin (target: "18")
- PWA support with auto-update and service worker
- Path alias: `@` → `./src`
- Manual chunk splitting: `arco`, `highlight`, `react`
- Build output: `build/` directory
- Dev/preview server: port 3000, host 0.0.0.0

### Pre-build Scripts
- `src/scripts/version-info.js` runs before build (generates version metadata)

## Key Dependencies

### UI Framework
- `@arco-design/web-react` - Primary component library
- `@arco-design/color` - Color utilities
- Custom theming via CSS custom properties in `src/theme.css`

### Notable Libraries
- `nanostores` + `@nanostores/persistent` - State management with localStorage
- `react-router` v7 - Client-side routing
- `ofetch` - HTTP client with interceptors
- `framer-motion` - Animations
- `dayjs` - Date formatting
- `node-polyglot` - i18n
- `highlight.js` - Code syntax highlighting
- `react-syntax-highlighter` - Syntax highlighting in React
- `html-react-parser` - Parse HTML strings to React
- `virtua` - Virtual scrolling
- `yet-another-react-lightbox` - Image viewer
- `plyr` - Media player
- `hls.js` - HLS video streaming

## Testing
No test framework is currently configured in this project.

## Deployment Notes
- Static SPA requiring URL rewrite rules (all routes → `index.html`)
- See README.md for Nginx/Caddy configuration examples
- Pre-built files available on `gh-pages` branch
- Docker image: `electh/reactflux`

## Common Patterns

### Adding a New Setting
1. Add to `defaultValue` in `src/store/settingsState.js`
2. Update Settings UI in `src/components/Settings/`
3. Access via `useStore(settingsState)` or `getSettings(key)`

### Adding a New API Endpoint
1. Create function in appropriate `src/apis/*.js` file
2. Use `apiClient.get()/post()/put()` from `src/apis/ofetch.js`
3. Handle errors (401 auto-redirects to login)

### Adding a New Route/Page
1. Create component in `src/pages/`
2. Add to `pageRoutes` object in `src/routes.jsx` (auto-generates list + detail routes)
3. Add navigation link in Sidebar component
