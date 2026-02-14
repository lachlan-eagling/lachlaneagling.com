# CLAUDE.md

Personal website — Astro 5, static output, zero client-side JS, strict TypeScript.

## Commands

- `npm run dev` — Dev server (localhost:4321)
- `npm run build` — Production build → `./dist/`
- `npm run preview` — Preview production build

## Project Structure

```
src/
  pages/          # File-based routing (.astro)
  layouts/        # Base HTML document, <slot /> composition
  components/     # Reusable .astro components
  assets/         # Images & files (optimized at build time)
public/           # Static files served as-is (favicon, robots.txt)
```

## Astro Components

- Define `interface Props {}` in the frontmatter; destructure with `const { ... } = Astro.props`
- Use `class:list={[]}` for conditional classes
- Use `<slot />` for composition — avoid wrapper `<div>`s when a semantic element or fragment works
- Conditional rendering: `{condition && <el />}` or ternary — no runtime JS
- Every page must use a layout: `<Layout title="...">`

## TypeScript

- No `@ts-ignore`, `@ts-expect-error`, or `as any`
- Use `import type` for type-only imports
- Prefer `const` over `let`; never use `var`

## Styling

- Scoped `<style>` tags in every component — no external CSS framework, no inline `style=`
- Use semantic HTML elements (`<nav>`, `<main>`, `<article>`, `<section>`, `<footer>`) over generic `<div>`
- Define shared values as CSS custom properties (`:root` in Layout)
- Use `rem`/`em` for sizing — avoid `px` except for borders and shadows
- Mobile-first: base styles for small screens, `min-width` media queries for larger

## Images & Assets

- Use `import` + `<Image />` from `astro:assets` for all content images (auto-optimized)
- Store images in `src/assets/` — only put non-optimizable files in `public/`
- Always provide `alt` text (empty `alt=""` only for decorative images)

## Accessibility

- All images require `alt` text
- Use landmark elements (`<header>`, `<nav>`, `<main>`, `<footer>`)
- Interactive elements must be keyboard-accessible
- Maintain sufficient colour contrast (WCAG AA)

## Content Collections

When adding blog/content:
- Define collections in `src/content.config.ts` with Zod schemas
- Query with `getCollection()` / `getEntry()`
- Generate pages with `getStaticPaths()` in dynamic route files (`[slug].astro`)

## Anti-Patterns — Do Not

- Add client-side JavaScript (`<script>` tags, framework islands) unless explicitly requested
- Install new npm packages without asking first
- Use inline `style=""` attributes — use scoped `<style>` instead
- Add unnecessary wrapper `<div>` elements — prefer semantic HTML or fragments
- Put optimisable images in `public/` — use `src/assets/` with `<Image />`
- Skip `alt` text on images
- Use `px` for font sizes or spacing
