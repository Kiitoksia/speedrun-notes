# Persona 5 Royal: Fusion Alarm RNG

How the game decides the random skills in an alarm fusion, whether that can be manipulated for speedruns, and what a
runner can do with it instead.

*PC version (Steam build 15515071). Findings from reverse engineering the executable plus in-game tests, November
2026. See also the [summary of fusion manipulation findings](fusion-manipulation.md).*

## The short version

| | Verdict |
|---|---|
| **Timing manip** | **No.** Even with the clock fixed and no input, two boots drift apart within a few frames, because load times vary. Confirming on a chosen frame can't pick the result. |
| **Fixed upgrades** | **Yes.** A known skill upgrade lands about 50–75% of the time on a persona with 1–3 skills. Reset until it does. |
| **Consoles** | **Untested.** Everything here is the PC version. |

## How the game's RNG works

- **The main generator** decides the alarm skills, and is also used by background visual effects and much else. It is
  seeded once, at boot, from the PC clock (date and time to the second), and isn't stored in save data, so loading a
  save doesn't reset it.
- **It is used constantly.** On the fusion confirm screen the game takes about 18 numbers per frame from it, in
  irregular bursts of 0 to 70+.
- **There is a second game generator** (a Mersenne Twister variant), used in about 430 places. It is also seeded from
  the PC clock, at startup and again from title-screen code. It doesn't decide alarm skills, and it wasn't used at all
  in the first 7,200 frames of a boot.

## How alarm skills are decided

Everything happens in one frame, about 500 numbers after you confirm the fusion (the fusion animation uses the
rest). The game:

1. **Shuffles the result persona's 8 skill slots** to decide which skills it looks at first. Inherited skills sit
   first, then the persona's own.
2. **Decides how many skills change.**
3. **Changes that many skills**, in shuffled order. For each one it tries six outcomes in a fixed order and stops at
   the first that succeeds.

### How many skills change

| Skills changed | Chance |
|---|---:|
| 1 | 20% |
| 2 | 60% |
| 3 | 20% |

### What each changed skill becomes

Skills have tiers 1–9. The outcomes are tried top to bottom; "when tried" is the chance that step succeeds if the game
gets to it.

| Outcome | When tried | Overall, skill has a fixed upgrade | Overall, it doesn't |
|---|---:|---:|---:|
| Random skill 3 tiers up | 1% | 1.0% | 1.0% |
| Random skill 2 tiers up | 5% | 5.0% | 5.0% |
| **The skill's fixed upgrade** | **80%** | **75.2%** | – |
| Random skill 1 tier up | 10% | 1.9% | 9.4% |
| Random skill 1 tier down | 30% | 5.1% | 25.4% |
| Random skill, same tier | always | 11.9% | 59.3% |

- "Random skill" picks evenly from every skill of the target tier, except ones the persona already has.
- A skill that can't go up far enough (for example a tier 7 skill can't go 3 tiers up) skips those outcomes and falls
  to the lower ones.
- Skills with no fixed upgrade are where the "good skill turned useless" results come from: they land on a random
  same-tier skill 59% of the time.

### Skills an alarm fusion can never give

The game leaves these out of the random pool, and no fixed upgrade produces them.

| Skill | Tier |
|---|---:|
| Victory Cry | 8 |
| Magic Ability | 7 |
| Almighty Amp | 7 |
| Soul Chain | 6 |
| Almighty Boost | 5 |

## Can it be manipulated?

One number's difference changes the whole result, and neighbouring positions in the RNG give unrelated results, so a
manip would have to land on an exact position. Tests showed that isn't possible:

| Test | Result |
|---|---|
| Predicting the result from the RNG state | Correct in all 5 fusions checked, so the model of the roll is exact. |
| Same seed (clock forced to the same time), same save, 4 boots | Four different results. |
| RNG forced to an identical state on the confirm screen, hands off, 3 tries | Different numbers used every frame: 5,445 / 5,481 / 5,463 over 300 frames. |
| Same seed, no input at all, 2 boots, 30 fps | Split apart at frame 14 and were 91 numbers apart after 7,200 frames. |
| Game clock fixed (every generator seeded the same), no input, 2 boots | Second generator identical and unused; the main one still split apart at frame 6. |
| Same, with frame times and the code behind every number logged (120 fps) | The same 67 pieces of code took the same amounts of numbers in both boots, but 35 of them did it 1–7 frames apart, right after loading hitches. |
| Same, capped at 30 fps | No steadier: split at frame 14 on the same effect, 41 of 67 shifted (mostly 3–10 frames), 39 numbers apart after 7,200 frames. |
| Numbers used between confirming and the roll | 486 and 498 in two runs, both over the same 18 frames. |

### The cause: loading time

The first same-seed tests only fixed the main generator. They were redone with the game's clock itself fixed, so
every clock-seeded generator got the same seed, and still no input. The boots still split apart at frame 6.

Logging every frame's length and which code took each number showed what happens. In the first 1,200 frames, all 67
pieces of code that use the generator took exactly as many numbers in both boots; the game did the same things. Only
the frame they happened on differed, by 1 to 7 frames, and the shifts line up with loading hitches (frames of
27–50 ms in an otherwise steady 8.3 ms). When a load finishes a frame sooner, the effects and objects that start after
it are created sooner, and many of them take random numbers as soon as they're created.

Because every system shares one generator, one event moving by a frame changes which numbers everything after it
gets. Load times depend on the disk, the operating system and the CPU, so no two attempts match exactly, even with
identical inputs.

This is the key difference from Persona 4 Golden, where a reset and the same inputs reproduce the same timeline.
Setting the clock gives a known seed, but the timeline drifts away from there regardless of inputs.

## What a runner can do

### Use fixed upgrades and reset

If a skill is picked to change, it lands on its fixed upgrade 75% of the time. How often it gets picked depends on how
many skills the result persona has:

| Skills on the result persona | Chance that skill gets its fixed upgrade |
|---|---:|
| 1 | ~75% |
| 2 | ~68% |
| 3 | ~50% |
| 4 | ~38% |
| 8 | ~19% |

Approximate: the game's slot shuffle is slightly biased, so some slots are a little more or less likely.

### Which upgrades exist

Each fixed upgrade needs Joker to be at least a certain level (the number in brackets below). Below that level the
upgrade is skipped and the skill falls through to the random outcomes. The useful ones:

**Any level**

- Resist X → Null X → Repel X → Drain X (every element and Phys, one step per alarm)
- Endure → Enduring Soul
- Agidyne / Bufudyne / … → Maragidyne / Mabufudyne / …
- Growth 2 → Growth 3, Regenerate 2 → 3, Invigorate 2 → 3

**Joker level 30–45**

- Assault Dive → Gatling Blow (30) → Megaton Raid (40)
- Swift Strike → Deathbound (40)
- Tempest Slash → Myriad Slashes (40)
- Vicious Strike → Heat Wave (40)
- Megido → Megidola (35)
- Rising Slash → Deadly Fury (45)

**Joker level 55–65 (late game)**

- Megaton Raid → God's Hand
- Sword Dance → Brave Blade
- Megidola → Megidolaon
- Mediarahan → Salvation
- Deathbound → Agneyastra
- One-shot Kill → Riot Gun
- Matarukaja etc. → Heat Riser
- Matarunda etc. → Debilitate

Example: Repel Phys early means Resist Phys → Null Phys on one alarm fusion, then Null Phys → Repel Phys on another,
each about 50–75% with a small result persona. Brave Blade early isn't possible: its upgrade needs Joker at level 65, and
the random route is about 0.2% per changed skill.

### Skills with no fixed upgrade into them: Tetra Break

Some skills, like Tetra Break, aren't the result of any fixed upgrade, so they can only come from the random outcomes.
There's no level requirement, but the odds are low. Tetra Break is tier 7, and the tier 7 random pool has 37 skills,
so a "random tier 7 skill" is Tetra Break about 1 time in 37 (slightly better, since the persona's own skills are
excluded).

Chance that one changed skill becomes Tetra Break, by the tier of the skill being changed:

| Skill being changed | Route to tier 7 | Chance per changed skill |
|---|---|---:|
| Tier 4 | 3 tiers up (1%) | ~0.03% |
| Tier 5 | 2 tiers up (5%) | ~0.13% |
| Tier 6, no fixed upgrade | 1 tier up (~9.5%) | ~0.26% |
| Tier 6, with fixed upgrade | 1 tier up (~1.9%) | ~0.05% |
| **Tier 7, no fixed upgrade** | **same tier (~63%)** | **~1.75%** |
| Tier 7, with fixed upgrade | same tier (~12.6%) | ~0.35% |
| Tier 8, no fixed upgrade | 1 tier down (30%) | ~0.8% |

A tier 6 skill can't go 3 tiers up (the tier 9 pool is empty), and a tier 7 skill can't go up 2 or 3, so the game
skips those outcomes without rolling and the lower ones come up more often than in the general table.

Best setup: a result persona with a single skill that is tier 7 with no fixed upgrade. That skill is always the one
changed, and becomes Tetra Break about 1.75% of the time, around 57 resets on average. Several such skills on the
persona give several chances per fusion (about 3.4% with three of them). These odds come from the same model that
predicted every tested fusion correctly, but this case hasn't been checked in game.

## Open questions

- **Consoles.** Untested. A quick check: set the console clock to a fixed time, boot, load the same alarm save and fuse
  the same way, five times. If any two results match, the console version may be deterministic and worth a closer
  look.
- **What triggers the alarm.** A percentage roll whose chance grows with each counted event, and the first alarm is
  guaranteed after a set count. Which event counts isn't confirmed yet.
- **Exact odds per fusion.** An offline calculator (no game reading, fine for runs) could give exact odds for a given
  persona and inheritance, including the shuffle bias. Not built yet.

## Tools used

A debug-only Reloaded-II mod (RNG Tracer) logs the game's random number calls, forces a seed and tests determinism. It
is a research tool, not something for runs.
