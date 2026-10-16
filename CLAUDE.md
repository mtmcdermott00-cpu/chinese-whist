# Chinese Whist scorekeeper

A phone-first scorekeeping app for a family card game the family calls Chinese Whist. It is an Oh Hell style bidding game. The app is a static site on GitHub Pages, installed to an iPhone home screen and run full screen (standalone web app). There is no server and no build step.

## Files

| File | Purpose |
| --- | --- |
| `index.html` | The whole app: markup, CSS and vanilla JS in one file. No framework, no dependencies. |
| `manifest.webmanifest` | Web app manifest: name "Chinese Whist", short name "Whist", standalone, portrait, black theme. |
| `sw.js` | Service worker for offline use. Network-first for the page, cache-first for icons and fonts. |
| `icon-180.png`, `icon-192.png`, `icon-512.png` | Home-screen icons: black ground, gold hairline frame, gold diamond between two rules. 512 doubles as the maskable icon. |

All paths are relative so the site works under `https://mtmcdermott00-cpu.github.io/chinese-whist/`.

## Game rules the app enforces

- 2 to 5 players, entered in clockwise seating order. One is marked as first dealer.
- Ten hands: 10, 9, 8 ... 1 cards. Trumps cycle Hearts, Diamonds, Clubs, Spades, No trumps, then repeat. So 10 Hearts, 9 Diamonds, 8 Clubs, 7 Spades, 6 No trumps, 5 Hearts, 4 Diamonds, 3 Clubs, 2 Spades, 1 No trumps.
- The deal moves one seat clockwise each hand. Bidding starts left of the dealer and the dealer bids last.
- Hook rule: the dealer may not bid the number that would make all bids total the cards dealt. The app blocks "Lock the bids" until that holds.
- Tricks entered must total the cards dealt before a hand can be scored.
- Scoring (house rules, editable on the setup screen): `points = tricks won x perTrick + (tricks won == bid ? bonus : 0)`. Defaults: perTrick 1, bonus 10.
- Optional bowers house rule (off by default): right bower (jack of trumps) and left bower (other jack of the same colour) are the top two trumps. Affects the Card ranks reference only, not scoring.
- After hand 10 the game is filed in the ledger. "Another round" starts a rematch with the next player dealing first.

## Screens (all in `index.html`, chosen by `S.view`)

- `setup`: masthead ("Chinese" in tracked gold caps over italic "Whist"), seats, the ten-hands grid, house rules (per trick, bonus, bowers switch), "Deal me in".
- `game`: hand kicker with a Ranks pill, headline (e.g. "8 cards" plus suit), undo / ledger / new game icons, the ledger grid (one column per player, one row per hand, running total with bid/won underneath, gold for an exact call, muted rose for a miss), and one gold action button. Bids and tricks are entered in a bottom sheet in bidding order.
- `history` (the Ledger): standings (games, wins, average, % called exactly), list of finished games with remove (two-step confirm), Back up / Restore.
- `ref` (Card ranks): pick a trump suit or no trumps, see trumps as cards high to low, the plain-suit order, and a follow-suit reminder. Opens on the current hand's trumps.

## State and storage

- One state object `S`. `S.game` = `{players, firstDealer, phase: setup|bid|tricks|over, round, bids[], tricks[], history[{bids, tricks}], startedAt, savedId}`. `S.rules` = `{perTrick, bonus, bowers}`.
- Saved to `localStorage` on every change: `cw.state.v1` (game, rules, updatedAt) and `cw.past.v1` (finished games, newest first). Every read and write is wrapped in try/catch.
- Finished game record: `{id: "g<startedAt>", finishedAt, startedAt, players[], totals[], hits[], rounds[], rules}`.
- Backup exports `{app: "chinese-whist", version: 1, exportedAt, state, past}` as JSON (Web Share sheet on iOS, download elsewhere). Restore merges games by id and only adopts the saved game if the current one hasn't started.
- Data lives on the phone only. Nothing is sent anywhere. Keep it that way unless the owner asks for sync.
- Changing the storage shape: bump the key suffix and migrate from the old key, never silently drop saved games.

## Design system

Black-tie casino look. Keep it consistent.

- Colours (CSS tokens on `:root`): ink `#0B0A09`, panel `#13110E`, gold `#C9A45C`, bright gold `#E2C27E`, ivory `#F3EBDD`, soft `#CFC6B6`, muted `#A39A8B`, dim `#6E665A`, red suits `#E07A86`, miss `#B9727B`. Single dark theme by design.
- Type: Bodoni Moda (italic for headlines and big numbers), Jost for body and small tracked uppercase labels. Loaded from Google Fonts with fallbacks.
- Gold hairlines and a thin gold frame inset around the screen, square 2px buttons with tracked uppercase labels, 44px minimum touch targets.
- Suit glyphs use U+FE0E so iOS doesn't render them as emoji.
- Copy tone: short, a little old-casino ("Deal me in", "Place the bids", "Settle the hand", "Cash out", "The house salutes"). No em dashes in UI copy.

## Standalone / iPhone details

- `viewport-fit=cover` and `apple-mobile-web-app-status-bar-style: black-translucent`, so the app runs under the status bar. `body` is padded by `env(safe-area-inset-top/bottom)`, the bottom sheet adds the bottom inset.
- `html, body` don't scroll; inner `.scroll` regions do. `overscroll-behavior: none` stops rubber-banding the whole app.
- Inputs are at least 16px so iOS doesn't zoom on focus.

## Shipping a change

1. Edit the files.
2. **Bump `CACHE` in `sw.js`** (e.g. `whist-v1` to `whist-v2`) whenever any file changes, so installed phones pick up the new version.
3. Commit to `main`. GitHub Pages deploys from `main`, root folder. Changes go live within a couple of minutes. On the phone, close and reopen the app (sometimes twice) to load the update.

## Testing

- Syntax-check the script: extract the `<script>` block and run `node --check`.
- Test in Safari on iPhone from the home-screen icon, not just a Safari tab: standalone mode, safe areas and storage behave differently there.
- Not yet verified on a real device: first launch from the icon, offline load, Back up via the share sheet, Restore from Files.

