# Persona 5 Royal: Glowing Personas and Fusion Accident Skill Sets

How the "fuse two glowing personas" trick works, what skill sets it can give, the real odds of each, and whether the
Nuke (Frei) Dyne set can be guaranteed.

*PC version (Steam build 15515071). Findings from reverse engineering the executable and the game's data files, plus an
in-game test, November 2026. See also [Fusion Alarm RNG](fusion-alarm-rng.md).*

## The short version

| | Verdict |
|---|---|
| **Can you guarantee the Frei Dyne set?** | **No.** It's one random pick with no player input. |
| **Odds of the Frei Dyne set** | **5.9%** (6 in 101) per fusion of two glowing personas. |
| **Odds of Freidyne on its own** | **14.9%** (15 in 101) per fusion of one glowing and one normal persona (no Mafreidyne). |
| **Timing manip** | **No.** The same random number generator as the alarm skills, which can't be timed on PC. |
| **Consoles** | **Untested.** |

## How it works

1. **The glow.** A persona you fuse during a Fusion Alarm gets a yellow name. The glow only lasts while that alarm is
   on: it's wiped from every persona when the alarm turns on or off.
2. **Glowing persona in = accident.** During an alarm, any fusion that uses a glowing persona is always a fusion
   accident. There's no roll for it.
   - The one exception: the very first alarm fusion of your save is always normal, because accidents only unlock after
     it.
3. **The accident persona is random.** You get a persona you don't own, with a base level between Joker's level -20
   and +5. It has its own starting skills plus the skills you picked to inherit for the fusion.
4. **Then a skill set is picked at random** from one of two tables:
   - **Both personas glowing:** the "Dyne set" table below.
   - **Only one glowing:** the "Boost/Evade" table below.
5. **The set replaces the persona's skills one for one.** The persona's first skill becomes the set's first skill, its
   second becomes the second, and so on. Skills the persona already has from the set are kept and skipped.
   - If the persona only has 4 skills, you only get the first 4 skills of the set. For the Nuke set those are
     Freidyne, Mafreidyne, Concentrate and Nuke Break.
   - **Inheriting a full set of skills into the fusion means you get more of the set.**
6. **Each set also gives a trait** (shown in the tables).

The in-game test matched this exactly: two glowing personas gave Jikokuten with 8 skills (3 of its own + 5 inherited),
and the game swapped all 8 for the healing set, as predicted.

## Both personas glowing: the Dyne set table

| Set | Chance | Trait | Skills, in order |
|---|---:|---|---|
| Hama/Mudo | 4.0% | Deathly Illness | Hamaon, Mudoon, Mahamaon, Mamudoon, Tetraja, Survival Trick, Hama Boost, Mudo Boost |
| Gun | 3.0% | Striking Weight | One-shot Kill, Triple Down, Charge, Life Aid, Auto-Mataru, Trigger Happy, Gun Boost, Gun Amp |
| All Amps | 13.9% | – | Fire, Ice, Wind, Elec, Nuke, Psy, Bless and Curse Amp |
| Fire | 5.9% | Heated Bloodline | Agidyne, Maragidyne, Concentrate, Fire Break, Fire Boost, Fire Amp, Burn Boost, Drain Fire |
| Ice | 5.9% | Frigid Bloodline | Bufudyne, Mabufudyne, Concentrate, Ice Break, Ice Boost, Ice Amp, Freeze Boost, Drain Ice |
| Wind | 5.9% | Wind Bloodline | Garudyne, Magarudyne, Concentrate, Wind Break, Wind Boost, Wind Amp, Attack Master, Drain Wind |
| Elec | 5.9% | Electric Bloodline | Ziodyne, Maziodyne, Concentrate, Elec Break, Elec Boost, Elec Amp, Shock Boost, Drain Elec |
| **Nuke** | **5.9%** | **Atomic Bloodline** | **Freidyne, Mafreidyne**, Concentrate, Nuke Break, Nuke Boost, Nuke Amp, Attack Master, Drain Nuke |
| Psy | 5.9% | Psychic Bloodline | Psiodyne, Mapsiodyne, Concentrate, Psy Break, Psy Boost, Psy Amp, Attack Master, Drain Psy |
| Bless | 5.9% | Blessed Bloodline | Kougaon, Makougaon, Mahamaon, Concentrate, Bless Boost, Bless Amp, Attack Master, Drain Bless |
| Curse | 5.9% | Cursed Bloodline | Eigaon, Maeigaon, Mamudoon, Concentrate, Curse Boost, Curse Amp, Attack Master, Drain Curse |
| Support | 14.9% | Mouth of Savoring | Spirit Drain, Abysmal Surge, Invigorate 1, 2 and 3, Life Aid, Heat Up, Despair Boost |
| Physical | 3.0% | Striking Weight | Megaton Raid, Deadly Fury, Heat Wave, Deathbound, Charge, Nocturnal Flash, Attack Master, Arms Master |
| Healing | 13.9% | Savior Bloodline | Mediarama, Diarahan, Energy Shower, Me Patra, Mabaisudi, Samarecarm, Dekunda, Divine Grace |

## Only one glowing: the Boost/Evade table

| Set | Chance | Trait | Skills, in order |
|---|---:|---|---|
| All Boosts | 15.8% | – | Fire, Ice, Wind, Elec, Psy, Nuke, Bless and Curse Boost |
| All Evades | 14.9% | Rare Antibody | Evade Fire, Ice, Wind, Elec, Nuke, Psy, Bless and Curse |
| Ailments | 9.9% | Foul Stench | Lullaby, Nocturnal Flash, Makajamaon, Wage War, Tentarafoo, Evil Smile, Brain Jack, Abysmal Surge |
| Fear/Drain | 9.9% | Gloomy Child | Bloodbath, Evil Touch, Evil Smile, Ghastly Wail, Stagnant Air, Spirit Drain, Fear Boost, Ambient Aid |
| Fire/Wind | 5.0% | Thermal Conduct | Maragion, Agidyne, Magarula, Garudyne, Stagnant Air, Fire Break, Burn Boost, Ambient Aid |
| Ice/Nuke | 5.0% | Cold-Blooded | Mabufula, Bufudyne, Mafreila, **Freidyne**, Stagnant Air, Ice Break, Freeze Boost, Ambient Aid |
| Elec/Nuke | 5.0% | Static Electricity | Mazionga, Ziodyne, Mafreila, **Freidyne**, Stagnant Air, Elec Break, Shock Boost, Ambient Aid |
| Counters | 9.9% | Retaliating Body | Oni-Kagura, Taunt, Wage War, Tetrakarn, Makarakarn, High Counter, Rage Boost, Ambient Aid |
| Regen | 14.9% | Gluttonmouth | Life Drain, Regenerate 1, 2 and 3, Life Aid, Heat Up, Endure, Fortify Spirit |
| Psy/Nuke | 5.0% | Skillful Technique | Mapsio, Psiodyne, Mafreila, **Freidyne**, Lullaby, Tentarafoo, Ailment Boost, Ambient Aid |
| Fist/Gun | 5.0% | Pinch Anchor | Miracle Punch, Swift Strike, Rebellion, Revolution, Thermopylae, Apt Pupil, Fortified Moxy, Adverse Resolve |

Freidyne is 4th in its three sets, so the accident persona needs at least 4 skills it can swap for Freidyne to land.

## Can the Frei set be guaranteed or manipulated?

**No.** The set is decided by one number from the game's main random number generator. That's the same generator as
the alarm bonus skills, and on PC it can't be timed: even with identical inputs it drifts apart from one boot to the
next, because load times vary (details in [Fusion Alarm RNG](fusion-alarm-rng.md)).

The only things a runner controls are:

- **Which table.** Both glowing gives the Dyne sets; one glowing gives the Boost/Evade sets.
- **How much of the set you get.** Inherit as many skills as you can so the accident persona has a full 8.

None of that changes which set comes up. It's retries only:

| Goal | Fusion | Chance per try | Average tries |
|---|---|---:|---:|
| Frei Dyne set (Freidyne + Mafreidyne + Nuke Boost/Amp/Break...) | glowing + glowing | 5.9% | ~17 |
| Freidyne (no Mafreidyne) | glowing + normal | 14.9% | ~7 |

## Things to know when planning

- **After a normal alarm fusion, the next fusion in the same alarm can be a forced accident.** The game remembers the
  alarm's last action and forces an accident when a normal fusion follows a normal fusion. This was seen once in game
  (two non-glowing personas fused second gave an accident with no roll), but a later run's second fusion was normal,
  so what resets it isn't pinned down yet. An accident from two non-glowing personas gives no skill set.
- **One alarm can give several tries.** Every glowing pair you fuse is a separate 5.9% roll, if you can make enough
  glowing personas while the alarm is on.
- **Leaving the Velvet Room ends the alarm and wipes the glow.** There's no saving between the fusions, so a failed
  roll means waiting for another alarm or resetting to before it.

## Open questions

- **What clears the forced accident.** The code points to doing a different kind of Velvet Room action in between
  (it looks like Gallows), but this hasn't been tested.
- **Whether an accident persona glows.** The code suggests every alarm fusion result glows, which would make an
  accident persona usable as the second glowing persona. Unconfirmed.
- **Consoles.** Untested.
