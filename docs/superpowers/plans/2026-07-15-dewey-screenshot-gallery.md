# Dewey Screenshot Gallery Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Add an App-Store-style, responsive, zero-JavaScript screenshot gallery directly below Dewey’s hero card without serving its original high-resolution screenshots.

**Architecture:** A dedicated `DeweyScreenshotGallery` Astro component owns the gallery markup, image transformations, and responsive CSS. The Dewey page eagerly imports the local screenshot directory, orders it by required two-digit filename prefixes, derives meaningful alt text from the required descriptive filename suffixes, and passes that metadata to the component. Astro’s local-asset pipeline generates only 480 px, 720 px, and 960 px WebP variants for delivery.

**Tech Stack:** Astro 5.17, `astro:assets` `Image`, scoped CSS, static-site output, npm.

## Global Constraints

- Store supplied screenshots in `src/assets/images/apps/dewey/screenshots/`; do not put them in `public/`.
- Do not add client-side JavaScript, a carousel library, or a package dependency.
- Preserve the App Store screenshot order exactly.
- Use local image imports and Astro `Image` for every gallery image.
- Produce only 480 px, 720 px, and 960 px wide WebP delivery variants at quality 80.
- Use mobile-first CSS with `rem` sizing and `min-width` media queries.
- The gallery must appear between the hero card and the existing `About Dewey` copy.

---

## File Structure

- Create: `src/assets/images/apps/dewey/screenshots/*.png` — the supplied App Store screenshots, renamed in their listing order using `01-`, `02-`, and subsequent two-digit numeric prefixes.
- Create: `src/components/DeweyScreenshotGallery.astro` — presentational, accessible horizontal gallery receiving image metadata from the page.
- Modify: `src/pages/apps/dewey.astro:2-6` — add the component import and typed local-asset glob.
- Modify: `src/pages/apps/dewey.astro:40` — insert the component before `.content-section`.

### Task 1: Prepare deterministic screenshot source assets

**Files:**
- Create: `src/assets/images/apps/dewey/screenshots/*.png`
- Modify: `src/pages/apps/dewey.astro:2-6`

**Interfaces:**
- Produces: An ordered `screenshots` array in `src/pages/apps/dewey.astro` whose items conform to `{ src: ImageMetadata; alt: string }`.
- Consumes: The user-supplied App Store screenshot files in their listing order.

- [ ] **Step 1: Create the asset directory and preserve the listing order in filenames**

  Put the supplied portrait screenshots in `src/assets/images/apps/dewey/screenshots/` and name them with consecutive two-digit prefixes. The filename suffix must describe the visible screen, for example `01-library.png`, `02-book-detail.png`, and `03-reading-session.png`. Keep the original file format at this stage; Astro will create the served WebP files.

- [ ] **Step 2: Inspect each source image before wiring it into the page**

  Run:

  ```bash
  sips -g pixelWidth -g pixelHeight -g format src/assets/images/apps/dewey/screenshots/*.png
  ```

  Expected: each asset reports a portrait pixel size and its source format. Record the visible Dewey screen represented by each image; use that wording as its alt text rather than using a filename or ordinal alone.

- [ ] **Step 3: Define typed page metadata from the numbered source filenames**

  Add this code below the existing `deweyIcon` import in the Dewey page frontmatter:

  ```ts
  import type { ImageMetadata } from "astro";

  interface DeweyScreenshot {
      src: ImageMetadata;
      alt: string;
  }

  const screenshotModules = import.meta.glob<{ default: ImageMetadata }>(
      "../../assets/images/apps/dewey/screenshots/*",
      { eager: true },
  );

  const screenshots: DeweyScreenshot[] = Object.entries(screenshotModules)
      .sort(([leftPath], [rightPath]) => leftPath.localeCompare(rightPath))
      .map(([path, module]) => {
          const label = path
              .replace(/^.*\//, "")
              .replace(/^\d+-/, "")
              .replace(/\.[^.]+$/, "")
              .replace(/[-_]/g, " ");

          return {
              src: module.default,
              alt: `Dewey ${label} screen`,
          };
      });
  ```

  The two-digit filename prefixes make alphabetical sorting identical to the App Store listing order. The descriptive filename suffix becomes the screen’s alt text, so choose it from the inspection in Step 2.

- [ ] **Step 4: Confirm sources are only build inputs, not public URLs**

  Run:

  ```bash
  git status --short src/assets/images/apps/dewey/screenshots src/pages/apps/dewey.astro
  ```

  Expected: the supplied screenshot files and the Dewey page metadata change are listed; no screenshot file exists under `public/`.

- [ ] **Step 5: Commit the asset preparation**

  ```bash
  git add src/assets/images/apps/dewey/screenshots src/pages/apps/dewey.astro
  git commit -m "assets: add Dewey app screenshots"
  ```

### Task 2: Build the reusable, native scroll-snap gallery

**Files:**
- Create: `src/components/DeweyScreenshotGallery.astro`

**Interfaces:**
- Consumes: `screenshots: Array<{ src: ImageMetadata; alt: string }>` from `src/pages/apps/dewey.astro`.
- Produces: A labelled section with a focusable native horizontal scroll region and responsive, optimised `img` elements.

- [ ] **Step 1: Write the component frontmatter and semantic markup**

  Create `src/components/DeweyScreenshotGallery.astro` with this frontmatter and markup:

  ```astro
  ---
  import { Image } from "astro:assets";
  import type { ImageMetadata } from "astro";

  interface Screenshot {
      src: ImageMetadata;
      alt: string;
  }

  interface Props {
      screenshots: Screenshot[];
  }

  const { screenshots } = Astro.props;
  ---

  <section class="screenshot-gallery" aria-labelledby="dewey-gallery-title">
      <header class="gallery-header">
          <h2 id="dewey-gallery-title">Dewey in action</h2>
          <p>Swipe or scroll to explore the app.</p>
      </header>
      <div
          class="gallery-scroller"
          tabindex="0"
          aria-label="Dewey app screenshots"
      >
          <ul class="screenshot-list">
              {
                  screenshots.map((screenshot, index) => (
                      <li class="screenshot-item">
                          <Image
                              src={screenshot.src}
                              alt={screenshot.alt}
                              class="screenshot-image"
                              widths={[480, 720, 960]}
                              sizes="(min-width: 37.5rem) 16rem, 78vw"
                              format="webp"
                              quality={80}
                              loading={index === 0 ? "eager" : "lazy"}
                              decoding="async"
                          />
                      </li>
                  ))
              }
          </ul>
      </div>
  </section>
  ```

- [ ] **Step 2: Add mobile-first gallery styling**

  Append this scoped CSS to the component:

  ```css
  <style>
      .screenshot-gallery {
          margin-bottom: 2rem;
      }

      .gallery-header {
          margin-bottom: 1rem;
      }

      .gallery-header h2 {
          font-family: var(--font-family-display);
          font-size: 1.5rem;
          font-weight: 600;
          letter-spacing: -0.02em;
          margin: 0 0 0.35rem;
      }

      .gallery-header p {
          color: var(--color-text-muted);
          font-family: var(--font-family-sans);
          font-size: 0.95rem;
          margin: 0;
      }

      .gallery-scroller {
          overflow-x: auto;
          overscroll-behavior-x: contain;
          padding: 0.25rem 0 0.85rem;
          scroll-snap-type: x mandatory;
          scrollbar-color: var(--color-border) transparent;
          scrollbar-width: thin;
      }

      .gallery-scroller:focus-visible {
          border-radius: 1rem;
          outline: 2px solid var(--color-text);
          outline-offset: 0.25rem;
      }

      .screenshot-list {
          display: flex;
          gap: 0.85rem;
          list-style: none;
          margin: 0;
          padding: 0;
      }

      .screenshot-item {
          flex: 0 0 78%;
          scroll-snap-align: start;
          scroll-snap-stop: always;
      }

      .screenshot-image {
          background: var(--color-bg);
          border: 1px solid var(--color-border);
          border-radius: 1.25rem;
          box-shadow: 0 0.75rem 2rem rgba(0, 0, 0, 0.12);
          display: block;
          height: auto;
          width: 100%;
      }

      @media (min-width: 37.5rem) {
          .screenshot-item {
              flex-basis: 16rem;
          }
      }

      @media (prefers-reduced-motion: reduce) {
          .gallery-scroller {
              scroll-behavior: auto;
          }
      }
  </style>
  ```

- [ ] **Step 3: Run Astro’s static type and component compilation check**

  Run:

  ```bash
  npm run build
  ```

  Expected: the build completes successfully and Astro reports no missing `alt` attribute or type error for the `screenshots` prop.

- [ ] **Step 4: Commit the gallery component**

  ```bash
  git add src/components/DeweyScreenshotGallery.astro
  git commit -m "feat: add Dewey screenshot gallery"
  ```

### Task 3: Place the gallery and verify generated assets and responsive behaviour

**Files:**
- Modify: `src/pages/apps/dewey.astro:2-6`
- Modify: `src/pages/apps/dewey.astro:40`

**Interfaces:**
- Consumes: `DeweyScreenshotGallery` and the ordered `screenshots` array from Tasks 1 and 2.
- Produces: `/apps/dewey/` with the gallery after the hero and before `About Dewey`.

- [ ] **Step 1: Add the gallery import and render it in the approved location**

  Add this import beneath the existing layout import:

  ```ts
  import DeweyScreenshotGallery from "../../components/DeweyScreenshotGallery.astro";
  ```

  Insert this component between the closing `</div>` for `.app-hero-card` and the opening `<div class="content-section">`:

  ```astro
  <DeweyScreenshotGallery screenshots={screenshots} />
  ```

- [ ] **Step 2: Run the production build and inspect the optimised output**

  Run:

  ```bash
  npm run build
  find dist/_astro -type f -name "*.webp" -print
  ```

  Expected: the Astro build succeeds and `dist/_astro` contains WebP files for the gallery in addition to the existing app-icon derivatives. The original supplied PNG files are absent from `dist/`.

- [ ] **Step 3: Perform viewport and interaction checks with JavaScript disabled**

  Run:

  ```bash
  npm run dev
  ```

  Expected: at a 375 px viewport, the first screenshot occupies about 78% of the gallery width and part of the second image is visible; at 700 px, 16 rem cards show roughly two to two-and-a-half images. Swiping or scrolling snaps to a screenshot’s left edge; the scroll region receives a visible outline on keyboard focus; all screenshots remain visible and scrollable with JavaScript disabled.

- [ ] **Step 4: Confirm source-to-delivery compression**

  Run:

  ```bash
  du -h src/assets/images/apps/dewey/screenshots/* dist/_astro/*.webp
  ```

  Expected: the original source files may exceed 1 MB, while the gallery’s generated WebP assets are substantially smaller and no full-size source image is served by the page.

- [ ] **Step 5: Commit page integration and verification-ready output**

  ```bash
  git add src/pages/apps/dewey.astro
  git commit -m "feat: display Dewey app screenshots"
  ```

## Plan Self-Review

- **Spec coverage:** Task 1 preserves the supplied App Store ordering and keeps source images under `src/assets`; Task 2 delivers the static, accessible, responsive gallery and configured WebP variants; Task 3 places it beneath the hero and checks build output, layout, scrolling, focus, and JavaScript-independent operation.
- **Placeholder scan:** The plan contains no deferred implementation marker. The supplied screenshot set is accommodated by the local-asset glob, with deterministic order and alt text supplied by the mandatory two-digit prefix and descriptive filename suffix.
- **Type consistency:** `ImageMetadata` and `{ src: ImageMetadata; alt: string }` are defined in both the page-facing contract and component, and `screenshots` is passed under that exact property name.
