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
- Bowers are always in (no switch): right bower (jack of trumps) and left bower (other jack of the same colour) are the top two trumps. `S.rules.bowers` is always true. Affects the Card ranks reference only, not scoring.
- After hand 10 the game is filed in the ledger. "Another round" starts a rematch with the next player dealing first.

## Screens (all in `index.html`, chosen by `S.view`)

- `setup`: masthead ("Chinese" in tracked gold caps over italic "Whist"), seats, the ten-hands grid, house rules (per trick, bonus, and a note that bowers are always in), "Deal me in".
- `game`: hand kicker with a Ranks pill, headline (e.g. "8 cards" plus suit), undo / ledger / new game icons with small labels (Undo, Ledger, New), the ledger grid (one column per player, one row per hand, running total with bid/won underneath, gold for an exact call, muted rose for a miss), and one gold action button. Bids and tricks are entered in a bottom sheet in bidding order. Tapping a player name in the ledger header opens a Names sheet to fix names mid-game without touching scores (also renames the filed record if the game is over).
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

Black-tie casino look, with a day version. Keep both consistent.

- Two themes from one token set. Night (dark) is the default on bare `:root`: ink `#0B0A09`, panel `#13110E`, gold `#C9A45C`, bright gold `#E2C27E`, ivory text `#F3EBDD`, red suits `#E07A86`, miss `#B9727B`. Day (light) redefines the same tokens under `prefers-color-scheme: light` and `[data-theme="light"]`: ground `#FBF9F4`, ink text `#17130E`, bronze gold `#86672B`, deep gold `#6A4F1A` for emphasis, red suits `#A3243A`.
- Never write a literal colour in a component rule. Use the tokens; translucent golds use `rgba(var(--gold-rgb), a)`. Text on a gold fill uses `--on-gold`. Card faces in the ranks reference use `--card`, `--card-ink`, `--card-red`, `--card-edge`.
- Appearance setting on the setup screen: Auto (follows the phone), Day, Night. Stored in `localStorage` as `cw.theme`, applied to `<html data-theme>` by a tiny script in the head so there's no flash, and mirrored into the `theme-color` metas.
- Type: Bodoni Moda (italic for headlines and big numbers), Jost for body and small tracked uppercase labels. Loaded from Google Fonts with fallbacks.
- Gold hairlines and a thin gold frame inset around the screen, square 2px buttons with tracked uppercase labels, 44px minimum touch targets.
- Suit glyphs use U+FE0E so iOS doesn't render them as emoji.
- Copy tone: short, a little old-casino ("Deal me in", "Place the bids", "Settle the hand", "Cash out", "The house salutes"). No em dashes in UI copy.

## Standalone / iPhone details

- `viewport-fit=cover` with `apple-mobile-web-app-status-bar-style: default`, so the status bar takes its colour from `theme-color` and its text stays readable in both themes. `body` is padded by `env(safe-area-inset-top/bottom)`, the bottom sheet adds the bottom inset.
- `html, body` don't scroll; inner `.scroll` regions do. `overscroll-behavior: none` stops rubber-banding the whole app.
- Inputs are at least 16px so iOS doesn't zoom on focus.

## Shipping a change

1. Edit the files.
2. **Bump `CACHE` in `sw.js`** (e.g. `whist-v1` to `whist-v2`) whenever any file changes, so installed phones pick up the new version.
3. Commit to `main`. GitHub Pages deploys from `main`, root folder. Changes go live within a couple of minutes. On the phone, close and reopen the app (sometimes twice) to load the update.

## Testing

- Syntax-check the scripts: extract both `<script>` blocks (the tiny theme one in the head and the app) and run `node --check` on each.
- Test in Safari on iPhone from the home-screen icon, not just a Safari tab: standalone mode, safe areas and storage behave differently there.
- Not yet verified on a real device: first launch from the icon, offline load, Back up via the share sheet, Restore from Files.

