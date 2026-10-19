# Teece ASCII art

Every card has a drawing in `CARD_ART` in `index.html`, keyed by card id with
tokens included. It sits behind the card face in hand, in a draft, in the Detail
panel and, for a Teece, inside its panel on the board. These are the rules for
drawing one.

(Battlegrounds also have a board drawing in `GROUND_ART`, drawn around the Teece
standing on them. Those have their own rules in the comment above `GROUND_ART`.
This file covers `CARD_ART`.)

## Size: as big as it needs, and shrunk to fit

There is no fixed size any more. The old rule of five rows by thirteen columns
is gone. A drawing can have any number of rows and any width. `artPre` pads
every row to the longest one, and the CSS (`.tart>.ascii`) picks the largest
type size that fits the whole drawing in its box. The drawing keeps its shape:
it is never cropped and never stretched. A drawing smaller than 13 by 5 is
treated as 13 by 5, so a small drawing never gets blown up.

Shrinking still costs something. The board is where a drawing has the least
room, and a phone is where that room is smallest:

| Where                         | Art box (CSS px) | Box shape in characters |
|-------------------------------|------------------|-------------------------|
| Board panel, phone 390 wide   | about 59 x 34    | about 3 columns per row |
| Board panel, desktop          | about 89 x 52    | about 3 columns per row |
| Hand card, square view, phone | about 84 x 53    | about 2.7 per row       |
| Draft card                    | about 168 x 50-95| wide                    |
| Detail panel                  | about 138 x 135  | about 1.75 per row      |

A character cell is about 0.6 wide by 1.04 tall in em, so a box that is 59 by
34 pixels holds about three columns for every row. **A drawing should be about
three times as many columns as it has rows.** A tall, narrow figure wastes most
of the box and comes out tiny.

The budget:

- **Teece (they stand on the board): at most 24 columns and 8 rows.** That keeps
  type at about 4px on a phone board panel and about 6px on a desktop one. The
  old 13 by 5 drawings render at 6.5px on a phone, so a full-budget drawing is
  roughly two-thirds the size of an old one, with three to four times the detail.
  A boss can go a little over (25 or 26 columns) if it needs to.
- **Magic, equips and battlegrounds (hand, draft and Detail only): up to about 34
  columns and 9 rows.** Their smallest box is a hand card, which is bigger than
  a board panel.
- **Tokens can stay small.** A three-row Spiderling or Bat is fine. It renders
  at the old 13 by 5 size.

The budget is not "five lines". It is "readable at the smallest place it is
shown".

## Style: one deck, one hand

A deck reads as a set when every drawing in it is built the same way. Decide
these things once per deck, write them below, and keep to them:

- **Line weight.** Single strokes (`/ \ | _`) or doubled ones (`// \\ ||`),
  rounded (`( ) . '`) or angular (`/ \ _ |`).
- **Faces.** How eyes are drawn (`==`, `o`, `@`, `>  <`, `..`) and what the
  mouth is (`VVVV` teeth, `vv` fangs, `< \`' >` mandibles). This is the
  deck's signature, so pick one and repeat it.
- **Bodies.** Front-on or in profile, and how a figure is built: a box skull
  with claws, a bandaged walker, a cape spread like wings, legs stacked in
  `/\` and `//\\` columns.
- **Characters the deck favours.** A short list of the deck's own marks: `=`
  and `-` for bandage, `#` for rot or armour, `~` for mist, `+` for holy.
- **Scale.** A rank-and-file Teece fills most of the budget. A boss fills all of
  it and has one extra feature: a crown, more eyes, a weapon, a hood.

**Draw the card, not the deck.** Every card shows what it does or what it is:
the Bone Thrower throws a bone, the Trapdoor Spider comes out from under a lid,
Soul Steal pulls a soul into a jar. The deck style is the handwriting, and the
card is the subject.

**No words, letters or numbers in a picture.** No `BOOM`, `AWOOO`, `R.I.P` or
`[R]`. The card already has a name.

## What good looks like, and what bad looks like

**Good** (these are in the game now):

- **The Priest deck (1102-1127).** Every priest is the same small figure: a
  round head `(o)`, a robe `/|=|\`, a base `/___\`. Each one adds one thing
  that says who it is: the Martyr's cross overhead, the Inquisitor's `?`, the
  Prophet's staff, the Seraph's wings. The `+` turns up on every card, so the
  deck is unmistakable at a glance. The Smites are just lightning, getting
  heavier from a Quarter to a Divine.
- **Ascetic (1111).** Five rows: a closed-eyed head over a seated body on a mat.
  It reads as "sitting still" with nothing extra in it.
- **Gun Range (1020).** Two targets on posts over a firing line. It is a picture
  of a place, not a symbol for one.
- **Hourglass Knight (385).** A helmet, an hourglass for a body, sand running
  out at the feet. The card's effect is the drawing.
- **The End Draws Near (777).** Two cards falling into a pit. Again, the effect
  is the picture.
- **Drone (TD).** A body with four rotors. One clear silhouette that is easy to
  read when it is small.

What they share: one subject, a clear outline, the deck's marks repeated,
and nothing in the picture that does not belong to the subject.

**Bad** (this is what the Zombie, Mummy, Vampire and Spider decks looked like
before they were redrawn):

- **One template wearing different hats.** Every Vampire and Zombie Teece was
  the same `( o o )` head with a `\ ~~ /` mouth over a stick body. Two cards
  differed by one character. Nothing said "vampire" (no cape, no collar, no
  fangs worth the name) or "zombie" (no rot, no claws, no teeth).
- **A box with eyes.** Every Mummy was the same `.=====.` rectangle with two
  eyes in it. That reads as a box with eyes. A mummy is a wrapped figure
  walking with its arms out, and none of them walked.
- **A stick figure with legs.** Every Spider was `\(o o)/` with a few legs
  under it, which is the same stick-man every other deck uses. A spider's whole
  shape is its legs, so the legs have to be the drawing.
- **Texture instead of a picture.** `:o::o:`, `#######`, `~ ~ ~ ~` fill the box
  and show nothing.

## The four redrawn decks, as worked examples

**Zombie.** A box skull drawn in single angular strokes (`/ \ | _`): `==` slit
eyes, a `\/` nose, a row of `VVVV` teeth over a `\^^^^/` jaw. Hands are
`|>|||` / `|||<|` claws with `VVVV` fingers, held up and reaching. Rot is `#`.
The skull is the subject. The body shows only when the card is about the body
(Rotting Brute, Bloated Zombie).

**Mummy.** A wrapped figure walking right in profile: a round head `.--.`, one
eye showing through the wraps (`|= o \`), the body built from `=` and `-`
bands, one stiff arm held straight out (`\_=-_-=_)`), feet `(__) \__)`, and a
loose bandage end trailing behind (`_.-`). Egypt comes in through props
(an ankh, canopic jars, a scarab, a pyramid, sand `.:`), and royalty through the
striped nemes headdress `|||` and the crook and flail.

**Vampire.** Front-on, inside a high collar that flares into a cape: the cape
is a big V with `\'-._` and `_.-'/` at the collar points, a scalloped hem
`/\/\/\`, a widow's peak `.-^^-.` with `/ \/\/ \` hair, eyes `>  <`, and fangs
`V  V`. Bats are always `/\_/\` ears, a `( o.o )` face, `vv` fangs and
scalloped wings `/\/\`. Blood is `'` and `,`, and mist is `~`.

**Spider.** The legs are the drawing: on each side they are stacked `/\`,
`//\\` and `//  \\` columns bending out and down, around a small head with `..`
eyes, a body `((  ))` and mandibles `< \`' >`. Spiderlings are the same in
miniature. The Web is radial (`\ | /`, `--+--`) with a ring of `-` around it.
Egg sacs are a round outline full of `o`.

## Checking that a drawing reads

1. **Write it as text, not as JS.** Draw in a plain text file and convert it
   with a script (JSON-encode each row), because hand-escaping `\\` in JS
   strings is where drawings get broken. Pad every row to the same width, which
   `artPre` also does.
2. **Count it.** A Teece is at most 24 by 8, or a boss about 26 by 8. Anything
   else is at most about 34 by 9. Aim for about three columns per row.
3. **Look at it at board size on a phone.** Serve the worktree
   (`npx http-server -p 8150 -s -c-1 .`), open a 390x844 viewport with a device
   scale of 2 or 3, put the cards on the board (`newState` plus `makeTeece`
   into `G.board[i].teece`, then `render()`), and screenshot the board. Then
   crop that screenshot at full resolution and look at it. If you cannot tell
   what it is, simplify it: drop the inner detail, keep the outline and the
   deck's signature marks.
4. **Check that it fits.** On every `.tart>.ascii`, `scrollWidth` must be no more
   than `clientWidth`, and the same for height. The CSS guarantees this, but a
   stray tab or a wide Unicode character breaks the measurement, so use plain
   ASCII.
5. **Look at the deck as a sheet.** Put all twenty cards side by side. If two
   look like the same drawing, or one looks like it came from another deck,
   redraw it.

Fusions stitch the left half of one drawing to the right half of the other
(`stitchArt`), centring the shorter one vertically. Any two drawings work.

## Deck style notes

One or two lines per deck: what the deck's drawings have in common now, and
where they fall short.

- **Basic.** A little soldier `_n_ (o_o) /|=|\ d b` in every role, with only
  the gear changing. Consistent. This is the template for a plain deck.
- **Rats.** One rat face `( o v o )` printed ten times on purpose, the King and
  Queen crowned. The spells (Swarm, Overwhelm, Pestilence) are texture, not
  pictures.
- **Berserker.** Horned helmets and `[###]` torsos, axes and blood. The spells
  and sigils are mixed and need one hand.
- **Rock.** Boulder bodies `( o o )` with `.-'-.` shoulders, and rubble `(_)`.
  Consistent. Bedrock and Clinging Moss are only texture.
- **Ogre.** One head `( o o ) (^\___/^)` repeated on every Teece. This is the
  same problem the old Vampire deck had.
- **Robot.** Square heads `[o_o]` and `|===|` chassis. Consistent and readable.
- **Gadget.** Boxes with dials and switches. Mixed. The fields (Polarity Field)
  are only lines.
- **Time.** Hourglass Knight is the model: draw time as hourglasses, clocks and
  sand. Fast Forward, Rewind, Pause and Time Warp are bare arrows and bars.
- **Witch.** Pointed hats `/\ /__\` over `(o o)` faces. The hexes are a mix of
  symbols.
- **Bomb.** Round bombs with fuses `.-'-.` and `(*)` blasts. The mines and
  fields are texture.
- **Nature.** Plants over a `~~~~` ground line. Soil and vines are `####`
  texture.
- **Pixie.** One sprite template `\( o )/ (/|\)` on every card, with only the
  face changing. It needs the kind of variety the Priests have.
- **Spider.** Redrawn: see above.
- **Vampire.** Redrawn: see above.
- **Zombie.** Redrawn: see above.
- **Tic-Tac-Toe.** X and O characters on stick figures and boxes. The Toerrains
  are dotted boxes.
- **Solar.** Suns `--( o )--` and moons. Mixed hands.
- **Army.** Helmeted soldiers `(o o)` with webbing. Grenadier Greg spells out
  `BOOM`, and the formations are only dots.
- **Garbage.** Grubby faces, bins and bags, `%#&` filth. Mostly consistent.
  Recycling Bins uses letters.
- **Apocalypse.** The Horsemen ride, and that is good. The rest is mixed:
  River of Blood, Wastes and Chainmail are texture.
- **Madness.** Faces with `@` and `?` eyes, but no shared construction.
- **Dimension.** Nested `+--+` boxes. It is abstract, and most cards look alike.
- **Locust.** Insect heads `\\ ^ //` with feelers. Consistent, but it leans on one
  template.
- **Devil.** Horned faces `/\ /\` and contracts. Consistent.
- **Runeblade.** Swords, with the letter `R` written into several of them.
- **Wizard.** Pointed hats with `*` stars over `(o o)` faces. The element tokens
  are simple glyphs. Earth and Deep Earth are `####` texture.
- **Gun.** Gunmen, ammo and plating. Mostly fine. Gun Range is a model ground.
- **Fusion.** Chimera bodies stitched from parts. No shared style yet.
- **Lightning.** Bolts and storm figures. Mixed, with several plain boxes.
- **Mummy.** Redrawn: see above.
- **Wolf.** Wolf heads with no single construction. Howl spells out `AWOOO`.
- **Priest.** The model deck: see above.
- **Dwarves.** `(o_o)` faces over `WWW` beards, under helmets. Consistent.
- **Warlock.** One small `\o/ /_\` warlock under a different patron sign on
  every card. Consistent.
- **Cult.** Hooded faces and pentacles. Mixed.
- **Paladin.** Great helms `_/^\_` with `|+++|` tabards. Consistent.
- **Knights.** Horse and rider in profile, `_[=]_` helms and lances.
  Consistent, and a good model for a deck of mounts.
