# lachlaneagling.com

Personal website for Lachlan Eagling — engineering leader, polyglot dev, indie builder.

Built with [Astro 5](https://astro.build), static output, zero client-side JS, strict TypeScript.

## Stack

- **Framework:** Astro 5 (static output)
- **Styling:** Scoped CSS, CSS custom properties, no external framework
- **TypeScript:** Strict, no `any`
- **Images:** `astro:assets` with `<Image />` for auto-optimisation

## Commands

| Command           | Action                                      |
| :---------------- | :------------------------------------------ |
| `npm install`     | Install dependencies                        |
| `npm run dev`     | Start dev server at `localhost:4321`        |
| `npm run build`   | Build production site to `./dist/`          |
| `npm run preview` | Preview production build locally            |

## Project Structure

```
src/
  pages/        # File-based routing (.astro)
  layouts/      # Base HTML document, <slot /> composition
  components/   # Reusable .astro components
  assets/       # Images & files (optimised at build time)
public/         # Static files served as-is (favicon, robots.txt)
```

## Links

- [lachlaneagling.com](https://lachlaneagling.com)
- [LinkedIn](https://www.linkedin.com/in/lachlan-eagling-44675a94/)
- [GitHub](https://github.com/lachlan-eagling)
- [Bluesky](https://bsky.app/profile/lachlan.bsky.social)
- [Mastodon](https://mastodon.social/@lachlane)
