# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a SvelteKit project named "iconify-max-depth-issue" - appears to be a minimal reproduction case for testing an Iconify-related issue. The project uses Svelte 5 with TypeScript and follows standard SvelteKit conventions.

## Common Commands

### Development
- `npm run dev` - Start development server
- `npm run dev -- --open` - Start dev server and open in browser

### Building & Preview
- `npm run build` - Create production build
- `npm run preview` - Preview production build locally

### Code Quality
- `npm run check` - Run Svelte type checking
- `npm run check:watch` - Run type checking in watch mode
- `npm run lint` - Run Prettier and ESLint checks
- `npm run format` - Format code with Prettier

### Setup
- `npm run prepare` - Sync SvelteKit (used for setup)

## Architecture

### Project Structure
- `src/routes/` - SvelteKit file-based routing
  - `+layout.svelte` - Root layout with favicon setup
  - `+page.svelte` - Homepage component
- `src/lib/` - Shared library code (imported via `$lib` alias)
  - `assets/` - Static assets like favicon
- `static/` - Public static files
- Standard SvelteKit configuration files at root

### Tech Stack
- **Framework**: SvelteKit with Svelte 5
- **Language**: TypeScript
- **Build Tool**: Vite
- **Linting**: ESLint with Prettier
- **Adapter**: Auto adapter for deployment

### Key Files
- `svelte.config.js` - SvelteKit configuration
- `vite.config.ts` - Vite configuration
- `tsconfig.json` - TypeScript configuration
- `eslint.config.js` - ESLint configuration

## Development Notes

- Uses Svelte 5 syntax (runes like `$props()`)
- File-based routing follows SvelteKit conventions
- Assets in `src/lib/assets/` can be imported via `$lib/assets/`
- No test framework is currently configured

## Issue Reproduction

This project reproduces the `effect_update_depth_exceeded` error from [Iconify GitHub issue #386](https://github.com/iconify/iconify/issues/386).

### Setup
- 3 horizontal PaneForge panes, each containing the same set of Iconify icons
- Configurable icon count via number input (defaults to 100)
- Icons are loaded on-demand from Iconify API (no local icon data)

### Error Threshold

#### Multiple Panes (3 panes, icons in each)
- ✅ **100 icons per pane (300 total)**: Works fine
- ❌ **500 icons per pane (1500 total)**: Triggers `effect_update_depth_exceeded` error
- ❌ **1000 icons per pane (3000 total)**: Triggers `effect_update_depth_exceeded` error

#### Single Pane - Incremental Loading (1 pane empty, 1 pane with all icons)
- ✅ **1500 icons in single pane**: Works fine when increased incrementally
- ✅ **2500 icons in single pane**: Works fine when increased incrementally  
- ✅ **3000 icons in single pane**: Works fine when increased incrementally

#### Single Pane - Initial Load (starting with high count from page load)
- ❌ **1000 icons in single pane**: Crashes on initial load
- ❌ **1500 icons in single pane**: Crashes on initial load
- ❌ **2000+ icons in single pane**: Crashes on initial load

**Critical Discovery**: The crash behavior depends on **HOW** the icons are loaded:
1. **Incremental increases**: Can handle 3000+ icons without issues
2. **Initial bulk load**: Crashes at ~1000 icons even in single pane

This suggests the issue is in Iconify's initial rendering/setup phase when many icons need to be processed simultaneously during component initialization.

### Error Details
```
effect_update_depth_exceeded
Maximum update depth exceeded. This typically indicates that an effect reads and writes the same piece of state
```

The error originates from `@iconify/svelte` at line 1763 in the `$effect` block, suggesting an infinite update loop in the icon loading/rendering logic when dealing with many icons simultaneously.