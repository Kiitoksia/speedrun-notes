# Persona 5 Royal: Fusion Manipulation Findings

What can and can't be manipulated in Fusion Alarm fusions, for two goals:

1. **Skill manipulation**: getting a chosen skill from the bonus skill changes of an alarm fusion.
2. **Skill set manipulation**: getting a chosen skill set (for example the Frei Dyne set) from fusing two glowing
   personas.

Section 3 covers a related trap: when an Electric Chair during an alarm can fail, and which order avoids it.

*PC version (Steam build 15515071). Findings from reverse engineering the executable and data files, plus in-game
tests, November 2026. Detail pages: [Fusion Alarm RNG](fusion-alarm-rng.md) and
[Glowing personas and fusion accident skill sets](fusion-accident-skill-sets.md).*

## The short version

| | Skill manipulation (alarm bonus skills) | Skill set manipulation (two glowing personas) |
|---|---|---|
| **Decided by** | Random rolls after the fusion animation | One random pick during the fusion |
| **By setup** | **Yes, partly.** A skill's fixed upgrade lands ~75% of the time on a one-skill result persona. | **No.** Setup only picks the table and how much of the set you receive. |
| **By timing** | **No** | **No** |
| **Plain odds** | Fixed upgrade ~75%; other targets usually under 2% | Frei Dyne set 5.9%; Freidyne alone 14.9% |
| **Consoles** | Untested | Untested |

## 1. Skill manipulation (alarm bonus skills)

When you fuse during an alarm, 1 to 3 of the result's skills change (20% / 60% / 20%). Each changed skill tries a
fixed list of outcomes in order. Full tables are on the [Fusion Alarm RNG](fusion-alarm-rng.md) page.

**What you can control: the setup.**

- **Fixed upgrades are the strong lever.** If a skill has a fixed upgrade and gets picked to change, it becomes that
  upgrade about 75% of the time.
- **Fewer skills on the result persona means your skill is more likely to be the one picked.**

| Skills on the result persona | Chance your skill gets its fixed upgrade |
|---|---:|
| 1 | ~75% |
| 2 | ~68% |
| 3 | ~50% |
| 4 | ~38% |
| 8 | ~19% |

- **Fixed upgrades need Joker to be a certain level.** Available at any level: Resist → Null → Repel → Drain (any
  element or Phys, one step per alarm), Endure → Enduring Soul, the -dyne spells → their Ma- versions, Growth 2 → 3,
  Regenerate 2 → 3, Invigorate 2 → 3. The big physical and almighty upgrades need Joker at level 30 to 65.
- **Skills with no fixed upgrade into them are long shots.** Example: Tetra Break can only come from the "random
  skill" outcomes. The best setup (a result persona whose only skill is tier 7 with no fixed upgrade) gives about
  1.75% per alarm fusion. Joker's level doesn't matter for it.

**What you can't control: the timing.** The rolls happen after the fusion animation, and the number of random numbers
the game uses between your confirm press and the rolls changed on every measured attempt. Confirming on a chosen
frame can't pick the result.

**So:** pick a target with a fixed upgrade, build a result persona with as few skills as possible, and reset if it
misses.

## 2. Skill set manipulation (two glowing personas)

**The mechanic** (details and full tables on the
[skill sets page](fusion-accident-skill-sets.md)):

- A persona fused during an alarm has a glowing (yellow) name while that alarm lasts.
- Fusing with a glowing persona during the alarm is always a fusion accident. The accident replaces the new persona's
  skills, one for one, with a whole skill set, and adds a trait.
- **Both personas glowing** picks from the Dyne-set table: each element's full set is 5.9% (Frei Dyne set:
  Freidyne, Mafreidyne, Concentrate, Nuke Break, Nuke Boost, Nuke Amp, Attack Master, Drain Nuke).
- **One glowing, one normal** picks from the Boost/Evade table: Freidyne (with Mafreila) appears in three sets,
  14.9% in total.

**Making the glowing pair.** Two fusions in a row during an alarm make the second one a forced accident, so put an
Electric Chair in between:

1. Fuse persona A (it glows).
2. Electric Chair any persona.
3. Fuse persona B (it glows).
4. Fuse A + B: the accident, and the skill-set roll.

In practice this is **one roll per alarm**: the twins end the alarm afterwards (runner-observed).

**What setup controls:** which table (one or both glowing), and how many of the set's skills you receive (the
accident persona swaps as many skills as it has, in order, so inherit a full set of skills).

**What setup doesn't control: which set.** That is a single pick from the game's main random number generator.

### Can it be manipulated? No.

Every route was checked:

- **Boot-time (clock) manipulation.** The generator is seeded from the PC clock, but two boots with the same seed and
  no input drift apart within the first frames, because load times vary and effects take random numbers when they
  start.
- **Counting something on screen.** About 15 effects on the fusion screens share the generator, several of them
  invisible or changing every few frames. Nothing visible keeps track of it.
- **Using an earlier result**, such as the first fusion's bonus skills. The same inputs give different load delays and
  different effect timings afterwards, so it says nothing usable about the next fusion.
- **A reset point.** There isn't one: the room's effects run from the moment you enter, and the menu's effects start
  after loading-dependent delays (recorded twice with identical inputs, with different delays).

### What you can do

| Goal | Fusion | Chance per alarm |
|---|---|---:|
| Frei Dyne set | glowing + glowing | 5.9% |
| Freidyne (no Mafreidyne) | glowing + normal | 14.9% |

Reset odds only. Inherit a full set of skills so the accident persona can take the whole set.

## 3. Electric Chair during an alarm

An Electric Chair (itemize) during an alarm gives the persona's alarm item (Regent gives Regent R instead of Regent).
It can also have an accident and give a different item. The order of your alarm actions decides whether that can
happen.

**The rules.** The game remembers what your last alarm action was (a fusion or an Electric Chair).

| Your Electric Chair comes... | Result |
|---|---|
| first in the alarm (nothing fused or itemized yet) | **Always works.** No accident is possible. |
| right after a fusion | **A chance roll.** It may fail. |
| right after another Electric Chair | **Always fails.** |
| on a glowing persona (one fused during this alarm) | **Always fails.** |

Fusions work as before: a fusion right after an Electric Chair is always safe, and a fusion right after another
fusion is always an accident.

**The chance after a fusion.** It's the alarm's accident chance. It starts at 0% when the alarm starts and goes up
with what you do during the alarm: each persona bought from the compendium adds 25%, fusions can add more, and once
you've done two of the same action (two fusions, or two Electric Chairs) it gets an extra 50%. It can't go above
100%. A warning shows up once it reaches 90%. How much one fusion adds hasn't been measured yet, but it's more than 0
(a "fuse, then Electric Chair" has failed in a run).

**What you get when it fails.** One random item, 10% each, from a list picked by the persona's base level:

| Persona base level | Items |
|---|---|
| 1-35 | Incense (ST, EN, MA, AG or LU), Alluring Belt, Lucky Belt, Mighty Ring, Megido card, Divine Grace card |
| 36-55 | Hell Ring, Almighty Ring, Miracle Belt, Ambergris (ST, EN, MA, AG or LU), Heat Up card, Ailment Boost card |
| 56-99 | Deadly Fury Belt, Nirvana (ST, EN, MA, AG or LU), Fortify Spirit card, Amrita Shower card, Angelic Grace card, Life Aid card |

A glowing persona gives a skill card instead: a resist, dodge, evade, boost or amp skill for some element.

A failed Electric Chair still counts as an Electric Chair, so a fusion right after it is still safe.

**What to do.** If you need one fusion and one Electric Chair in the same alarm, **Electric Chair first, then
fuse**. Both are then guaranteed: the Electric Chair is the alarm's first action, and the fusion comes right after an
Electric Chair. "Fuse, then Electric Chair" risks losing the alarm item.

In the glowing-pair setup from section 2, the Electric Chair in step 2 comes right after a fusion, so it can fail.
That doesn't hurt the setup (step 3 is still safe); only the item is lost.

## Open questions

- **Consoles.** Everything here is the PC version.
- **Gallows instead of Electric Chair** between the two setup fusions: untested.
- **How much one alarm fusion raises the accident chance**, which sets the odds of a "fuse, then Electric Chair"
  failure. Being measured.
- **Whether a failed Electric Chair counts toward the twins ending the alarm.** It triggers the same "accident" event
  as a fusion accident, but that hasn't been checked in game.
