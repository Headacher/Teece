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

`main` carries the same tree but **not the same commits**, and it drifts. A
session that starts there is at least working on the right codebase, but it is
working on an older one: the shipping branch has repeatedly been several builds
and a dozen commits ahead. Never assume the two agree — diff them.

So a task branch is where you work, and the shipping branch is where you land.
Finishing means merging into `claude/overclocker-fly-gadget-bugs-rvlhsg`, not
pushing somewhere and stopping. **This holds for a new deck by default.** A deck
that exists only on `main` is a deck nobody can play, and "add a deck" is never
complete until it is on the branch Pages builds. If you are told to push
somewhere else, push there too — but say plainly that the change is not live
until it reaches the shipping branch, and offer to take it there.

The merge is usually small — your one commit onto whatever landed meanwhile —
but it conflicts on `VERSION` every time, and on `CARDS` whenever both sides
added cards next to each other. Keep the shipping branch's number and bump it
past theirs; keep both sides' cards. Then re-run the tests on the MERGED tree:
the shipping branch has its own engine work that your changes have never been
tested against.

**Push to the branch the owner can actually see.** Said plainly, because it has
been said three times in this file and still gets missed: the only branch worth
landing on is the one serving https://headacher.github.io/Teece/. A task branch
is scaffolding. If you finish a change and it is not on
`claude/overclocker-fly-gadget-bugs-rvlhsg`, you have not finished it — go and
put it there, in the same session, without being asked again.

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

## Adding a deck

A deck is registered in seven places, and missing one of them fails quietly —
the cards exist and simply never reach a player. Take the next free block of
twenty ids (Wizard is 906–925, with its conjured tokens at 926–941, the
tutorial's cards sit at 960–970, Gun is 1001–1020, and Fusion is 1021–1040),
then:

- `CARDS` in `index.html` — the twenty cards, plus any token cards they make.
  A token carries `token:true` and `deck:'<key>'`, which keeps it out of drafts
  while still colouring it as yours.
- `decks.json` — a `decks.<key>` entry with `name`, `cards`, `blurb` and `how`,
  and the twenty ids appended to `pool`. Edit it with a script rather than by
  hand; it is long enough that a stray comma is hard to see.
- `CARD_ART` — one 13-column drawing per card, tokens included. Anything not 13
  wide skews the face it sits behind.
- `DECK_SYM` — the deck's emoji.
- Three CSS blocks keyed on `data-card-deck="<key>"`: the `.card`/`.dcard`
  palette with its `.kwl`/`.ct` accent, the `::after` symbol, and `.teece` for
  the board. The rules name `.card` and `.dcard` together so a card in hand and
  the same card in a draft never diverge.

Then check it the way a player would: the deck appears in the picker, its cards
carry the palette in hand, and every one of the twenty actually fires in a real
game — drive some AI games and grep the log for each card's name, because a
card the AI never plays is a card you never tested.

And land it on the shipping branch. See the top of this file: a deck that stops
at `main` is a deck nobody can play.

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

A fresh browser context is a first-time player: the page skips the menu and
starts the interactive tutorial, which refuses every move but the one it is
teaching. A test that is not about the tutorial must mark it seen before the
page loads:

```js
await context.addInitScript(()=>{try{localStorage.setItem('teece.tutorial','seen');}catch(e){}});
```

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
- A rule that belongs to a DECK rather than a card goes in `decks.json` and is
  copied onto the seat at setup — `perTurn` and `freeDiscard` both work this
  way. There are two setup paths that do the copying (the two-seat one and the
  three-seat one) and `newState` is not one of them, so a headless test or a
  soak built straight on `newState` never sees the rule and will report the
  feature dead. Set the seat flag by hand in the harness.
- `opt.hands` lets a cell prompt accept a card in hand as the answer, but
  `aiPickCell` only ever looks at the board, so the computer and every headless
  prompt will ignore those targets entirely. Anything the AI has to be able to
  choose needs its own branch, the way `chooseSacrifice` does.
- A keyword's engine name and what a player reads can differ: `KW_LABEL` maps
  one to the other (`steadfast` reads "Immune to stat reduction"), and the
  Detail panel and deck pages show keywords through `kwLabel`. Card text should
  use the player-facing words.
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

### The tutorial is a scripted game, and it leans on the engine

The interactive tutorial (`startTutorial`) is a real game against a Trainer
whose five moves are fixed in `TUT_OPP`, played with effect-free cards 960–970
that are in no deck and not in the pool.

**The pool is not what the roguelike drafts from.** `allDraftable` walks every
card in `CARDS` and skips only `token`, `perkOnly` and `tutorial` cards (the
classic, Gadget and Garbage cards are drafted that way on purpose). So any card
that must never reach a player's deck needs one of those flags, or it turns up
in roguelike drafts: the tutorial's 14/14/14/14 no-sacrifice Warlord did,
builds 99–107, until the cards were given `tutorial:true`. The Witches'
random `conjure` walks every magic card the same way and skips tokens and
tutorial cards. Each of your turns is one lesson in
`TUT_LESSONS`; the current step is the first whose `done()` is false, read off
the board every render. The step gates three places: `playFromHand` (which card),
`chooseCell` (which panels or hand cards a prompt will take, via `tutFilter`)
and `endTurn`. Nothing else is special-cased, so a change to fights, equips,
plating, sacrifice, magic targeting or the domination check can quietly break a
lesson — the captions also do the arithmetic out loud ("10 beats 9"). The whole
script is also balanced so no one reaches a lead of four before the last lesson.
After touching any of those, play the tutorial through in a browser: a fresh
context, then keep clicking the centre of the one `.tuthi` element (by
coordinates, checking it sits above `#coach`) until `G.over`; it takes 23 taps
and must end on "Tutorial complete" on desktop and a phone viewport alike.

`TUT_PAGES` are the rules pages behind the menu's Rules button, drawn with the
classic cards by id. They quote numbers the engine keeps elsewhere (`WIN_LEAD`,
five cards to open, six to keep); re-read them when those change.

### Hands are hidden, and a developer code shows them

`handVisible(p)` decides whose cards a hand row draws: against the computer only
yours, at a hotseat only the seat whose turn it is (plus any seat a prompt is
asking right now). Every other hand is dealt face down. Typing up up down down
left right left right toggles `SHOW_ALL_HANDS` to show every hand. A page test
that reads the computer's hand from the DOM must turn that on first; `G` always
has the real hands.

Animation lengths live in `ANIM_MS`, not at the call sites, and the CSS
keyframes (`placePop`, `strikeFlare`, `dieShake`, `zapPulse`) are matched to
them. A fight is two beats in `resolveCombat`: the killers `strike`, then the
losers go `dying`.

### The Wizard conjures, combines and leaves weather behind

Wizards conjure element tokens (Fire 926, Water 927, Wind 928, Earth 929)
straight into the hand through `conjureEl`; each element's spell is the
`element` fx, which reads `archmageFor` for the stronger version. Two different
elements combine through `combineOpts`/`combineAt` into the tokens in
`EL_COMBO`, from a Combine button that takes the Discard button's place;
Alchemical Compounds is a wildcard (`alchemy`), and Holy needs a Grand Magus
(`holyCombine`) on the field. A magic card's `elem` list is what the
Elementals' `onMagicElem` reads, so Steam counts as Fire and as Water.

The Fireball's fire is a `scorch` fire: it gives no standing penalty, and
`scorchFires` (run at the top of every `resolveCombat`) takes -4 off whatever
stands in it for good and puts that fire out. The Sandstorm moves what stands
on it in `sandstormTurn`, at the start of every turn. A Ley Line pays on
arrival through `groundArrive`, called from `playTeece` and `fireMove`.

Balance, measured in AI-vs-AI games against every other deck: with bodies
around 5s it won 85% (Runeblade, the strongest other deck, wins 70% of the
same matchups), and the conjuring was the source. The owner then took 3 off
every side of all twelve Teece, bosses included, and it won 21% (build 103).
Settled at 2 off every side and 1 back on the Archmage and Grand Magus
(8s and 11s): 51% (build 104). Re-soak it after any change to the elements or
the bodies.

### The Gun spends bullets, six shots a turn

Bullets are numbers on the seat (`bullets`, plus Speedloader's `hotBullets`,
which are spent first and lapse in `endTurn`), not cards. `shots` counts the
magazine: `MAG_SIZE` is 6, `startTurn` refills it for the seat whose turn it
is, and `reloadMag` refills it early. Any seat holding bullets has a gun, so a
Warzone arms the other side and a drafted Gun card brings the gun with it;
`hasGun` only decides whether the chip and the count beside the name show.

The gun is the `gun` status chip under the board: `drawGun` opens a cell
prompt with `opt.gun` (which turns the pointer over the board into a
crosshair) and keeps reopening it until it is holstered or runs dry. Every
shot, from anywhere, goes through `shootTeece`, and that is where all of the
"when shot" cards answer — `shotGain`, `shotKw`, `onShotAny`, `shotStun`,
`shotExec` and the Warzone's `gShotBullet`. `fireGunAt` is the paid-for pull of
the trigger (a bullet, a shot, then `resolveCombat`). A Gun Range is shot by
raising `gmod` on the ground instance itself. The Stuntman's dodge is
`gunDodge`, called from `processDeath`; the Gun Devil's eight lines are
`gunLine`. The Avatar of the Gun God (`gunGod`) turns the chip off for the seat
that bolted it on and gives its wearer an act, `gunGodShot`.

The computer shoots in `aiGun`, one best shot at a time on a cloned board via
`simShot`, and keeps three bullets back while it holds a Gun Devil. Balance in
AI-vs-AI games against every other deck: 59% over 500 games (build 115).

### Fusion makes cards that nobody printed

Every Fusion Teece prints the `fuse` keyword. Two Teece cards in hand (one of
them a fuser, the other any Teece) fuse through `fuseHandOpts`/`fuseHandAt`,
from a Fuse button that takes the Discard button's place; two of a seat's own
Teece standing side by side fuse through the `kfuse` act (`FUSE_ACT`, which
`actsOf` adds to any body whose OWN card prints `fuse`), once per turn per
seat via `onceTurn:'fuse'`.

`makeFusionCard(a,b)` builds the result as a real card and registers it in
`CARDS` under a string id (`FU1`, `FU2`, ...) with `fusion:[a,b]` and
`deck:'fusion'`: sides and Defence added, keywords pooled (minus `fuse`, which
is why a fusion never fuses again), lists such as `onPlay`/`onDeath`/`act`
joined, numbers added, a second aura pushed into `auras`, art stitched from
both halves. Everything downstream therefore reads it as any other card. The
same pair always returns the same id (`FUSE_CACHE`), so the AI simulating a
fusion on a cloned board does not leave cards behind, and `newState` clears
them all (`forgetFusions`) — a fusion card made before a `newState` in a test
is gone after it. `toGrave` buries a fusion as its parts, recursively.

On the board `fuseOnBoard` adds the two bodies' own bases (so turns and
handicaps carry), perm/temp/temp2, Defence, keywords, equips, links and
grafts. `afterFusion` then breaks any Unstable Catalyst (`grants.fuseBreak`,
also checked on hand plating) and offers the Fusion Chamber's draw
(`gFuseDraw`). Hyperfusion (`fuHyper`) fuses two fusions from hand, field or
one of each into a card with `boss:true`, which `isBossCard` now also reads.
Phase Drift parks its +5 on `t.fuseCharge`, which `fuseOnBoard` turns
permanent. The Paradox Twins' "cannot win until the start of your next turn"
is a `nowin` hex marked `liftAtStart`, removed in `startTurn` rather than
counted down in `tickHexes`.

The computer fuses in hand in `aiFuseHand` (strongest pair, up to twice a
turn, before it plays a Teece), on the board when `aiActMuts` scores it, and
casts the three spells through `aiFusionMagicPlan`. Fusing is the deck: a
computer that never fused in hand won 24%; fusing every pair it can, up to two
a turn, won 47.5% and 50.5% in two 1040-game AI-vs-AI soaks against every
other deck (build 116).

### Games end at turn 100

`checkTurnLimit` runs at the end of every turn, after the other win checks:
when turn `TURN_LIMIT` (100, counted across all seats the way the turn banner
counts) ends, `deckOutWinner` decides it -- most Teece on the field, Tokens
not counted, level is a draw. The turn panel shows "turn N of 100" for the last
ten, and `lastTurnNotice` stops a human seat at the start of its final turn
with a notice it must acknowledge (not the computer's, not the tutorial's). A
page test that runs a game near turn 100 has to press `#lastTurnOk`. A soak that
loops "up to 300 turns" now always stops by 100.

### Day and Night exist only with Solar cards

The round turns the sky only when `skyShowing()` finds a card that reads it
somewhere in the game (board, hands, decks, graves); otherwise the phase stays
Day and nothing reads it. The Stopped Sundial (`timeStop`) stops only that
round's turn: `setPhase` still lets a card change the sky under it. A magic card
with `onlyWhen:'night'` refuses to be played at the wrong time, and a
`graveAct` with `when` is only offered in that half of the cycle. `defender` is
a keyword `scanDeaths` reads: that Teece never destroys anything in combat.

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
