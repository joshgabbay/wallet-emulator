# Wallet Emulator — Context

A static HTML wallet/ticket UI emulating an Apple Wallet style ticket carousel. Hosted on GitHub Pages from the `gh-pages` branch of `joshgabbay/wallet-emulator`. Live at https://joshgabbay.github.io/wallet-emulator/.

## Files

- `index.html` — single-page app. Contains the ticket carousel **and** the edit panel (overlay). All assets (banner, NFC icon, TM icon, hold-near-reader image) are inlined as base64. This is the page meant to be added to the iOS home screen.
- `edit.html` — legacy URL. Redirects to `index.html#edit` so old links/bookmarks land on the integrated editor.
- `CONTEXT.md` — this file.

## Data model

Tickets are stored in `localStorage` under the key `wallet_tickets` as a JSON array. Each ticket:

```js
{ time, date, venue, event, section, row, seat, entry, type }
```

If the key is missing, `defaultTickets` (defined at the top of the inline script) is used. Reset clears the key.

## Architecture

- View carousel and edit panel are in the **same** page so they share one localStorage scope.
- "..." button (top-right) opens the edit panel. "Done" closes it. "Save" writes to localStorage and re-renders the carousel.
- `index.html#edit` opens the panel on load (used by the `edit.html` redirect).
- Carousel uses native CSS scroll-snap (`scroll-snap-type: x mandatory`). Dots track scroll position via the scroll listener.

## Why edit + view live in one page (important)

iOS gives each "Add to Home Screen" entry its **own isolated localStorage**, separate from Safari and from any other home-screen app. Previously edit.html and index.html were separate pages: editing in Safari (or in an edit.html PWA) didn't reach the index.html PWA on the home screen because they were different storage partitions.

By keeping edit and view in `index.html`, the home-screen PWA reads and writes the same localStorage, so edits made inside the PWA persist across opens. **Always edit from inside the home-screen app**, not from Safari, or the home-screen view won't see the changes.

## PWA install

Meta tags already set:
- `apple-mobile-web-app-capable=yes`
- `apple-mobile-web-app-status-bar-style=black-translucent`
- viewport with `viewport-fit=cover` for safe-area insets.

To install: open the deployed `index.html` URL in Safari → Share → Add to Home Screen.

## Deploy

The repo's only branch in use is `gh-pages` — pushes to `origin/gh-pages` go live on GitHub Pages. There is no build step; commit the edited HTML directly.

```sh
git add index.html edit.html CONTEXT.md
git commit -m "..."
git push origin gh-pages
```

## Repo layout note

The user's working directory is `~/Developer/wallet-emulator/`, which contains older copies of `index.html` and `wallet.html` outside any git repo. The actual git checkout lives in `~/Developer/wallet-emulator/deploy/` and tracks `origin/gh-pages`. Edit files inside `deploy/`.
