# Teece

A card/board game in one self-contained file: `teece (4).html`. Cards, rules, AI,
rendering and all styling live in that file's single `<script>` and `<style>`
block. There is no build step and no dependencies — open the file and it runs.

## Bump the version on every push

`VERSION` sits near the top of the script and is printed on the start screen next
to the title, so a player can always name the build in front of them.

```js
const VERSION={build:1,date:'2026-09-05'};
```

**Every push must carry a version bump, in the same commit as the change:**

- `build` — increment by one. One bump per push, not per commit.
- `date` — the day of that push, `YYYY-MM-DD`.

If a change reaches `main`, the start screen has to name it. A push that leaves
`VERSION` untouched is incomplete. When several commits go out in one push, bump
once, in the last of them.

## Testing

The file exports its internals for a headless harness when `module` exists, so
the whole game can be driven from Node without a browser:

```js
if(typeof module!=='undefined'){ module.exports={newState,playFromHand,...}; }
```

Extract the script and require it:

```js
const L=require('fs').readFileSync('teece (4).html','utf8').split('\n');
const a=L.findIndex(s=>s.trim()==='<script>'), b=L.findIndex(s=>s.trim()==='</script>');
require('fs').writeFileSync('game.js', L.slice(a+1,b).join('\n'));
```

Then `newState(...)` / `setG(...)` / `getG()` to build a position, and
`playFromHand`, `runAct`, `endTurn`, `resolveCombat` to drive it. Set
`G.aiSeats=[]` to drive both sides by hand so no AI turn races an assertion.

Two things to know about the harness:

- With no DOM, every prompt auto-resolves through `aiPickCell`/`aiPickOpt`, which
  pick **randomly**. A test that depends on where a Teece lands must either
  constrain the board to one legal choice or retry until it gets the placement it
  needs.
- `HAS_DOM` is false, so `anim()` returns immediately and `toast()` is a no-op.
  Timing-sensitive behaviour has to be checked in a real browser instead.

For anything that touches rendering, input or overlays, drive the real page with
Playwright (Chromium is preinstalled at `/opt/pw-browsers`; never run
`playwright install`). Top-level `let` bindings like `G` are reachable from
`page.evaluate` as bare identifiers — `window.G` is not.

## House rules for the game code

- Card definitions are data in `CARDS`, keyed by id. Prefer adding a field the
  engine reads over special-casing a card by id.
- `effSides` is the single source of truth for a Teece's current numbers; auras,
  equips, grounds and doubling all compose there.
- Effects that must land before combat go on the effect stack via `queueFx`;
  `drainFx` empties it at the top of every `resolveCombat`.
- Hand entries are plain card ids indexed by position, with equip plating in a
  parallel `plate` array. Only `handPush` and `handTake` may touch either, or the
  two drift apart.
