# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Personal website built with Astro 5. Static site generation with zero client-side JavaScript by default (Astro's island architecture).

## Commands

- `npm run dev` — Start dev server on localhost:4321
- `npm run build` — Production build to `./dist/`
- `npm run preview` — Preview production build locally

## Architecture

- **Framework**: Astro 5 with strict TypeScript (`astro/tsconfigs/strict`)
- **Routing**: File-based routing via `src/pages/`
- **Layouts**: `src/layouts/Layout.astro` — base HTML document with `<slot />` composition
- **Components**: `src/components/` — `.astro` components with scoped `<style>` tags
- **Assets**: `src/assets/` for optimized imports; `public/` for static files served as-is
- **Styling**: Scoped CSS within components, no external CSS framework
- **Output**: Static HTML (no SSR adapter configured)
