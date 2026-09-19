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

Three things about the harness:

- With no DOM, every prompt auto-resolves through `aiPickCell`/`aiPickOpt`, which
  pick **randomly**. A test depending on where a Teece lands must constrain the
  board to one legal choice or retry until it gets the placement it needs.
- `HAS_DOM` is false, so `anim()` returns at once and `toast()` does nothing.
- A prompt offered with `skip:true` is **declined** whenever the placement does
  not move `boardValue` by 0.05 — `aiPickCell` weighs the board and takes the
  skip. Anything whose worth is later rather than now scores zero, so it never
  fires in a test and never fires for the AI: the Eldritch Library laying
  itself down read as a dead effect for exactly this reason. Before believing
  an effect is broken, check whether it offered a skip at all. If the card says
  "play this" rather than "you may", do not pass `skip`.

For rendering, input or overlays, drive the real page with Playwright — but serve
it over HTTP first (`npx http-server -p 8137 -s -c-1 .`), because `fetch('decks.json')`
needs a real origin and fails on `file://`. Chromium is preinstalled at
`/opt/pw-browsers`; never run `playwright install`. Top-level `let` bindings like
`G` and `DECKS` are reachable from `page.evaluate` as bare identifiers.

`-c-1` is not optional. Without it http-server sends `max-age=3600`, so a server
started before an edit keeps serving the old file and the failures it produces
look like real bugs in whatever you just wrote.

Three more things a page test gets wrong before it gets right:

- `G.turn` flips to the next seat inside `endTurn`, **before** `startTurn` has
  drawn, scanned or rendered. Waiting on the seat alone hands you control while
  all of that is still to come, and it stomps whatever board the test seeded.
  The turn banner (`--- Turn N: ---`) is the last line `startTurn` logs, so
  `G.turn===0 && !G.pending && /^--- Turn/.test(G.log[0])` is the honest signal.
- `boundingBox()` is viewport-relative, and a click past the fold lands on
  nothing at all — silently. That is the same bug as the phone one below, and it
  bites on a 1280x900 desktop too, because the hand sits at the bottom of a page
  taller than the window. Scroll it into the window and re-measure before
  clicking, on every viewport.
- On screen and inside the window is still not reachable: something can be
  sitting on top. Check `elementFromPoint` at the coordinates you are about to
  click, and print the element chain when it is not what you aimed at — a
  faded-out `#toast` swallowed a status chip on a phone exactly once in five
  runs before it was given `pointer-events:none`, and a one-in-five failure
  reads like a flake rather than the bug it is.

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

### An ability reads as X : Y ; Z

X is what turns it on, Y is what it costs, Z is what it does. Keep the three
apart when you write a card, because cards talk about each other in those terms
— the Necronomicon force-activates Z and never pays Y.

Y is marked in the data: a step with `cost:true` is part of the price, and a
`cost:true` step that reports NONE stops the list so the rest is never paid for
nothing. Put a `{fx:'price', …}` gate ahead of a list that takes more than one
thing, so an unaffordable ability costs nothing at all rather than eating the
first half of its price and stopping.

### Once a game means one of two different things

`once:'game'` is once per BODY — it lives on the Teece in `actUsedGame`, and
`makeTeece` hands a returning Teece a clean sheet. `onceGame:'<name>'` is once
per PLAYER for the whole game, kept on the seat in `onceUsed` under the name the
card picks, so two abilities can share one charge and nothing gives it back.

### A card in a graveyard can still have a button

`actsOf` reads a Teece off the board, so an ability printed on a card that is in
the ground has nowhere to be tapped. `grants.graveAct` is the list a card offers
from inside a graveyard; `graveActsFor` gathers them and `statusesFor` turns
each one into a status chip that *is* the button — a status pushed with a `run`
fires on the tap instead of only explaining itself.

An equip with `undyingRide` is not lying loose in the graveyard pile: it goes
down wrapped around the card it was bolted to and waits in `pl.riding` under
that card's name, in one place and one place only. Do not also push it onto
`grave` — `attachRiders` takes it out of `riding` without touching the pile, so
a card in both comes back on the Teece and stays buried at the same time.
