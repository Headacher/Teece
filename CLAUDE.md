# Teece

A card/board game in `index.html` — one self-contained file holding cards,
rules, AI, rendering and styling — plus `decks.json`, which lists the decks,
formats and draft pool. No build step.

## Check which branch you are on before you touch anything

**GitHub Pages serves `claude/overclocker-fly-gadget-bugs-rvlhsg`** at
https://headacher.github.io/Teece/. That branch, and only that branch, reaches a
player. Run `git rev-parse --abbrev-ref HEAD`. If it does not say that, switch:

```
git checkout claude/overclocker-fly-gadget-bugs-rvlhsg
```

This has already cost three sessions' work. Twice, nine commits went to `main`
and vanished. The third time, a session started on `main`, spent a day adding a
whole deck plus a UI pass to a file called `teece (4).html` — an older,
differently-numbered codebase that `main` used to carry — and every push
succeeded while none of it reached the game. The pushes are never the problem.
The branch is.

`main` now carries this same tree, so a session that starts there is at least
working on the right code; it still has to move to the shipping branch before
pushing, because Pages does not build `main`.

Before believing anything is live, check the deployment rather than the push. The
Pages build shows up as a `pages build and deployment` workflow run, and its
`head_branch` tells you what is actually being served.

Card ids on the retired `teece (4).html` lineage do not match this one — Nature
is 521–540 here and 501–520 there, and Witch (491–509), Pixie (541–560) and
Spider (561–580) do not exist there at all. Never port a change by card id
without checking the name.

## Bump the version on every push

`VERSION` sits near the top of the script and prints on the start screen beside
the title, so a player can always name the build in front of them.

```js
const VERSION={build:3,date:'2026-09-05'};
```

Every push carries a bump, in the same commit as the change: `build` up by one
(once per push, not per commit), `date` set to the day of the push.

## Testing

The file exports its internals when `module` exists, so the game can be driven
from Node. Because the decks live in `decks.json`, a headless harness has to feed
them in by hand:

```js
const src=require('fs').readFileSync('index.html','utf8');
const a=src.indexOf('<script>'), b=src.lastIndexOf('</script>');
require('fs').writeFileSync('/tmp/livegame.js', src.slice(a+8,b));
const g=require('/tmp/livegame.js');
g.setDeckData(JSON.parse(require('fs').readFileSync('decks.json','utf8')));
```

Then `newState` / `setG` / `getG` to build a position, and `playFromHand`,
`runAct`, `endTurn`, `resolveCombat` to drive it. Set `G.aiSeats=[]` to drive
both sides by hand so no AI turn races an assertion.

Two things about the harness:

- With no DOM, every prompt auto-resolves through `aiPickCell`/`aiPickOpt`, which
  pick **randomly**. A test depending on where a Teece lands must constrain the
  board to one legal choice or retry until it gets the placement it needs.
- `HAS_DOM` is false, so `anim()` returns at once and `toast()` does nothing.

For rendering, input or overlays, drive the real page with Playwright — but serve
it over HTTP first (`npx http-server -p 8137 -s .`), because `fetch('decks.json')`
needs a real origin and fails on `file://`. Chromium is preinstalled at
`/opt/pw-browsers`; never run `playwright install`. Top-level `let` bindings like
`G` and `DECKS` are reachable from `page.evaluate` as bare identifiers.

### "It doesn't work on mobile" usually means "I can't reach it"

Three separate reports — a pixie that could not fly, then the same again, then
Spin a trick — were all one bug: the control was rendered, enabled and working,
and sitting off the bottom or top of a phone screen. Nothing in the game state
looks wrong, so a test that builds a position with `page.evaluate` and clicks by
selector will pass every time while the real page is unusable.

To catch this class, a phone test has to fail the way a thumb fails:

- Drive the actual flow (draft → handicap → play), on a phone viewport, rather
  than assembling the board through `evaluate`.
- Never click by selector. Playwright's `click`/`tap` scroll the element into
  view first, which is the exact bug hiding itself. Read `boundingBox()`, check
  it lies inside `innerHeight`, and click the coordinates with `page.mouse`.
- Check the whole loop: the control on screen, the question it opens on screen,
  and a destination cell not covered by whatever just pinned itself.

Anything pinned to a viewport edge needs matching room held open at that end of
the page, or it buries the content underneath. On a phone `aside` stacks below
`main`, so the end of the page is the side panel — pad `#app`, not `main`.

## House rules for the game code

- Cards are data in `CARDS`, keyed by id. Prefer adding a field the engine reads
  over special-casing a card by id.
- `effSides` is the single source of truth for a Teece's current numbers; auras,
  equips, grounds and doubling all compose there.
- Effects that must land before combat go on the effect stack via `queueFx`;
  `drainFx` empties it at the top of every `resolveCombat`.
- Hand entries are plain card ids indexed by position, with equip plating in a
  parallel `plate` array. Only `handPush` and `handTake` may touch either, or the
  two drift apart.
- A card in hand and the same card in a draft both render from `CARD_ART` and
  `data-card-deck`; the deck palette rules name `.card` and `.dcard` together so
  the two never diverge.
