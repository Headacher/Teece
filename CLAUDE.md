# Teece

A card/board game in `index.html` — one self-contained file holding cards,
rules, AI, rendering and styling — plus `decks.json`, which lists the decks,
formats and draft pool. No build step.

## THIS is the branch that ships

**GitHub Pages serves this branch**, `claude/overclocker-fly-gadget-bugs-rvlhsg`,
at https://headacher.github.io/Teece/. It does **not** serve `main`.

`main` is a stale fork carrying an older, differently-numbered codebase in a file
called `teece (4).html`. The two have no shared history. Work merged to `main`
never reaches players, and this has already cost two sessions' work — once in
early September, and again in the session that wrote this file, where nine
commits went to `main` and vanished.

So: **do the work here.** Before believing anything is live, check the deployment
rather than the push — the Pages build shows up as a `pages build and deployment`
workflow run, and its `head_branch` tells you what is actually being served.

Card ids differ between the two branches. Nature is 521–540 here (Witch holds
491–509), but 501–520 on `main`. Never port a change by card id without checking
the name.

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
