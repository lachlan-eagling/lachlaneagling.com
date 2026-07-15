# Dewey Screenshot Gallery Design

## Goal

Show Dewey's App Store screenshots in an efficient, accessible gallery that
appears immediately below the app hero and communicates the product before the
page's explanatory copy.

## Scope

- Add a screenshot gallery to the Dewey app page only.
- Use the complete App Store screenshot sequence, in its existing order.
- Keep the page static: do not add a carousel library, client-side script, or
  package dependency.

## Layout and Interaction

The gallery will be inserted between the Dewey hero card and the existing
`About Dewey` section. It will have a short heading, such as “Dewey in action”,
and a horizontally scrolling sequence of portrait screenshots.

The sequence uses native browser scrolling with CSS scroll snapping. On narrow
screens, one screenshot is nearly full width and the next image is partially
visible to indicate that the content can be swiped. On larger screens, each
card is sized so that roughly two to two-and-a-half screenshots are visible.
This supports touch swipes, trackpads, mouse-wheel horizontal scrolling, the
browser scrollbar, and keyboard scrolling without JavaScript controls.

## Assets and Delivery

The supplied App Store screenshots will be stored as source files in
`src/assets/images/apps/dewey/screenshots/`. The page will import each file and
render it with Astro's `Image` component rather than referencing `public/`
assets.

Astro will create responsive WebP derivatives during the production build. The
gallery will request only the size appropriate to its rendered card width,
using approximately 640 px and 960 px wide variants at quality 80. The
original screenshots may remain larger than 1 MB as build inputs; they will
not be served directly. The first image loads eagerly enough to avoid an empty
gallery at the top of the page, while later images load lazily.

## Accessibility

- Use a labelled `section` containing a semantic list and make the scroll
  region keyboard-focusable with `tabindex="0"`.
- Give every screenshot concise alt text naming the shown Dewey screen.
- Preserve the browser-visible horizontal scrollbar and add a non-textual
  scroll cue through the partially visible next card.
- Give the focused scroll region a visible outline without introducing custom
  gallery controls or a custom keyboard model.
- Respect reduced-motion preferences by disabling smooth scrolling and
  entrance motion for the gallery.

## Visual Treatment

Screenshot cards will retain their native portrait ratio, have a modest border
and rounded corners that fit Dewey's existing glass-card styling, and use a
subtle shadow for separation from the page background. The gallery should not
be nested inside a prose card: it needs the available page width for the
App-Store-like horizontal composition.

## Verification

- Run the Astro production build successfully.
- Inspect the generated page at a narrow mobile viewport and a desktop
  viewport, confirming the intended visible-card counts and scroll snapping.
- Confirm generated build assets are WebP responsive derivatives rather than
  direct copies of the supplied screenshots.
- Check that all rendered `img` elements have appropriate alt text and that
  the gallery remains usable with JavaScript disabled.
