# Zombie Word Zapper: RPG Design Spec (v1)

> **Update (round numbers for kids):** the implemented values supersede the numbers below. Every number a kid sees is a whole, round number (multiples of 10); odd values need a real reason.
> - **HP:** base 100, Pumpkin Armor +20/tier, Juice Box +40, Snack Pack +1/tier per zap. No Easy start bonus, no per-level HP.
> - **Typo:** Easy 1, Medium 5, Hard 10 (floor 1 everywhere). Easy is gentler through forgiveness: 2 free oops per zap and at most 2 charged (Medium: 1 free, cap 3; Hard: none, no cap).
> - **Blaster jam:** typing is ignored for `D.jamT` seconds (Easy 1.5, Medium 2, Hard 2.5). It jams after `D.jamStreak` wrong keys in a row (Easy 6, Medium 5, Hard 4; a right key resets the streak, and the oops shield doesn't stop it). Keys pressed while jammed are ignored and never count as typos. **Zombies charge while the blaster is jammed:** walking zombies move faster (`D.charge`: Easy 2x, Medium 3x, Hard 4x) (lean forward, groan, kick up dust) until it unjams. Freeze still stops them. **On the last heart it breaks instead** (every difficulty; Easy and Medium get there more slowly thanks to their free oops, grace and cap): the blaster throbs red, smokes and sparks as a warning, and the next charged typo (the oops shield, grace window and cap still protect; the break beats the streak jam) breaks it for good. There's a "BROKEN!" badge with no bar, items stop working, no new stages start, and zombies charge at `D.charge`, like any jam, just without end, so the next bite ends the run. Visuals: a "JAMMED!" badge with a draining bar, a greyed-out keyboard helper, and a drooping, rattling, smoking blaster with a red gem.
> - **Bite / boss bite:** Easy 10 / 20, Medium 20 / 40, Hard 30 / 60 (a boss bite is always 2 bites).
> - **Heals (stage / perfect word / perfect sentence):** Easy 30 / 2 / 10, Medium 20 / 2 / 4, Hard none. The only 1s are base units (typo 1 on Easy, Snack Pack +1).
> - **Coins:** 1 per zap (2 for big words); +10 for a 10-streak, boss, perfect sentence, stage clear, and 90% accuracy; score ÷ 200. Multipliers Easy ×1, Medium ×1.5, Hard ×2, magnet +25%/tier, shown on results as whole bonus coins. New save starts with 40 coins + 1 Juice Box.
> - **Prices:** Juice 40, Freeze 60, Bomb 80. Armor 100/300/600, Oops 100/400, Magnet 200/400/800, Splash/Sleepy/Snack 200/600. Blasters 100, Gold 500.
> - `DIFFICULTY` / `RPG` / `UPGRADES` / `ITEMS` / `BLASTERS` in index.html are the source of truth.


Owner: game design. Implementers: developer (logic) and graphics artist (UI/3D/FX).
Scope: persistent coins, a shop, one HP bar with gentle typo damage, and light XP levels.
Everything stays inside `index.html`. All numbers live in one `RPG` constants object so balancing is a one-line change.

Design pillars for ages 6 to 11:
1. **Typos sting, but they never end a run.** Only zombies can knock you out.
2. **Every run pays.** Even a short, bad run earns enough coins to buy something small.
3. **One bar, one currency, three power-up buttons.** No menus inside menus.

---

## 0. Difficulty presets (replaces Zombie Speed)

The menu's "Zombie speed" fieldset (Sleepy / Shuffly / Speedy) becomes **Difficulty** with the options Easy / Medium / Hard. Each option is a bundle of settings. `G.speed` is renamed `G.diff`, and every rule that depended on speed now reads `D = DIFFICULTY[G.diff]`. The old `SPEEDS` table and the `G.speed === "slow" / "fast"` factors in `maxAlive`, `spawnInterval` and the sentence spawn timer are removed.

```js
const DIFFICULTY = {
  easy: {
    speed: 0.85,          // replaces SPEEDS[...]; old values were slow .95 / medium 1.4 / fast 1.95
    spawnK: 1.25,         // multiplies spawnInterval() and the sentence token spawn gap
    maxAlive: 6,          // maxAlive(n) = Math.min(2 + n, D.maxAlive)
    typoDmg: 0.5,
    freeOops: 2,          // free typos after each zap (+ Oops Shield tiers)
    grace: 0.8,           // seconds after a charged typo when more typos are free (0 = off)
    typoCap: 3,           // max charged typos between zaps (Infinity = no cap)
    typoFloor: 1,         // typos can never take HP below this
    bite: 20, bossBite: 30,
    healStage: 35, healPerfect: 2, healPerfectSentence: 8,
    regen: true,          // Snack Pack upgrade works
    startHpBonus: 20,     // added to maxHp for the run
    coinMult: 1.0,
    scoreMult: 1.0
  },
  medium: {
    speed: 1.35, spawnK: 1.0, maxAlive: 8,
    typoDmg: 1, freeOops: 1, grace: 0.5, typoCap: 3, typoFloor: 1,
    bite: 25, bossBite: 40,
    healStage: 25, healPerfect: 1, healPerfectSentence: 5,
    regen: true, startHpBonus: 0,
    coinMult: 1.25, scoreMult: 1.0
  },
  hard: {
    speed: 1.95, spawnK: 0.85, maxAlive: 8,
    typoDmg: 1, freeOops: 0, grace: 0, typoCap: Infinity, typoFloor: 1,
    bite: 30, bossBite: 50,
    healStage: 0, healPerfect: 0, healPerfectSentence: 0,
    regen: false,         // only the Juice Box heals on Hard
    startHpBonus: 0,
    coinMult: 1.75, scoreMult: 1.0
  }
};
```

| Knob | Easy | Medium | Hard |
|---|---|---|---|
| Zombie speed (base) | 0.85 | 1.35 | 1.95 |
| Spawn gap factor / max alive | ×1.25 / 6 | ×1.0 / 8 | ×0.85 / 8 |
| Typo damage | 0.5 | 1 | **1** (every misclick) |
| Free oops per zap / grace / cap | 2 / 0.8 s / 3 | 1 / 0.5 s / 3 | 0 / none / none |
| Typo can't drop HP below 1 | yes | yes | yes |
| Bite / boss bite | 20 / 30 | 25 / 40 | **30** / 50 |
| Heal: stage / perfect word / perfect sentence | 35 / 2 / 8 | 25 / 1 / 5 | **0 / 0 / 0** |
| Snack Pack regen | yes | yes | **no** |
| Start HP | 120 | 100 | 100 |
| Coins | ×1 | ×1.25 | ×1.75 |
| Score | ×1 | ×1 | ×1 |

Decisions and notes:
- **Hard stays strict but still kid-safe.** Every misclick costs exactly 1 HP: there is no free oops, no grace window and no cap. The typo floor of 1 still applies, so a typo alone can never end a run. At 1 HP one more typo breaks the blaster for good and the zombies charge (see the jam note at the top), so the typo leads to the knockout, but the bite still delivers it.
- **Upgrades on Hard:** things you buy still work. The Juice Box heals (the only healing on Hard). Oops Shield tiers still give free oops (`freeOops = D.freeOops + up.oops`), because blocking a typo is not healing. Pumpkin Armor, levels and Sleepy Dust apply everywhere. Only Snack Pack is off (`D.regen === false`). Its shop card shows "Doesn't work on Hard".
- **Score multiplier stays 1** everywhere, because each difficulty keeps its own best score (see below). The knob is kept for tuning.
- **HP rounding:** `G.hp` is a float. The bar width uses the exact value. The number shown is `Math.ceil(G.hp)`, so 99.5 shows as "100" and never shows 0 while you're still alive. Game over is checked with `G.hp <= 0`. Floating "-½" pops are shown on Easy.
- **Coins:** the old `SPEED_COIN` is replaced by `D.coinMult`.

**Menu labels** (radio `name="difficulty"`, values `easy|medium|hard`, ids `d-easy|d-medium|d-hard`):
- **Easy**: "Slow zombies, gentle oopsies"
- **Medium**: "Faster zombies, small ouches"
- **Hard**: "Speedy zombies, no healing, more coins!"

**Saved choice and migration:** the choice is stored under the new key **`zwz-diff`**. On load, if `zwz-diff` is missing, read the old `zwz-speed` and map it `{slow:"easy", medium:"medium", fast:"hard"}` (anything else becomes `"easy"`). Write `zwz-diff` and leave `zwz-speed` untouched. It is harmless and allows a rollback. Valid values are checked with `in DIFFICULTY`.

**Best scores:** each mode and difficulty pair has its own best, `zwz-best-<mode>-<diff>`. To read it: `store.get(newKey, null) ?? store.get("zwz-best-" + mode, 0)` (falling back to the old per-mode best until it is beaten). Only the new key is written. `refreshBestLine()` listens to changes on both the mode and difficulty radios and says "Best in Words · Hard: 1,240". Game over compares against the same fallback value.

**Developer checklist (difficulty)**
1. Add `DIFFICULTY`. Replace `SPEEDS[G.speed]` in `zombieSpeed()` with `D.speed * (1 - 0.06 * up.slow)`. In `maxAlive` use `D.maxAlive`, and in `spawnInterval` and the sentence spawn gap use `D.spawnK`.
2. Replace the speed fieldset with the difficulty fieldset and labels. Rename `G.speed` to `G.diff`. Migrate `zwz-speed` to `zwz-diff`.
3. Route every HP rule through `D` (sections 1 and 4a): typo damage and forgiveness, bites, heals, regen gate and start bonus.
4. Coins: `mult = D.coinMult * (1 + 0.2 * up.magnet)`.
5. Per-difficulty best keys with the old-key fallback, and a best line that updates on both radios.
6. HP display with `Math.ceil`, and fractional pops on Easy.

---

## 1. HP model (replaces hearts)

**Decision: HP replaces hearts.** A single HP bar replaces `#lives`, `G.lives` and `G.maxLives`. Two health systems would confuse young kids, and a bar can show small typo damage where hearts cannot. All damage, heal and forgiveness values come from `D = DIFFICULTY[G.diff]` (section 0).

| Constant | Value | Notes |
|---|---|---|
| `RPG.BASE_HP` | 100 | Start of every run: `G.hp = G.maxHp` |
| Bite damage | `D.bite` / `D.bossBite` | 20/30 on Easy, 25/40 on Medium, 30/50 on Hard |
| Typo damage | `D.typoDmg` | 0.5 / 1 / 1 |
| Forgiveness | `D.freeOops`, `D.grace`, `D.typoCap`, `D.typoFloor` | Hard: no free oops, no grace, no cap. The floor is 1 everywhere |
| Heals | `D.healStage`, `D.healPerfect`, `D.healPerfectSentence` | All 0 on Hard |
| `RPG.LOW_HP` | 34% of max | At or below this: bar pulses and a soft heartbeat plays |

**Max HP formula:** `G.maxHp = 100 + D.startHpBonus + 20 * up.maxhp + 2 * (level - 1)`
**Free oops:** `freeOops() = D.freeOops + up.oops`
**Heal helper:** `heal(n)` clamps to `G.maxHp`. Every non-Juice heal call site passes the `D.*` value, which is 0 on Hard. Snack Pack only heals when `D.regen` is true.

### Typo rules (exact order inside `miss(ch, z)`)
```js
function typoHurt(z) {
  const D = DIFFICULTY[G.diff];
  if (z) z.oops = true;                        // spoils the "perfect" heal for that zombie
  if (++G.typoStreak >= D.jamStreak) { jam(...); return; }        // key mashing jams the blaster
  if (G.oopsLeft > 0) { G.oopsLeft--; fxOopsBlocked(); return; }  // free oops
  if (G.hp <= D.typoFloor) { breakBlaster(); return; }          // at the floor one more typo breaks the blaster
  if (G.typoGrace > 0 || G.typoCharged >= D.typoCap) return;      // forgiveness (Hard: never)
  G.typoCharged++; G.typoGrace = D.grace;
  G.hp = Math.max(Math.min(G.hp, D.typoFloor), G.hp - D.typoDmg);
  fxTypoHit();
}
```
- On every zap (word completes, or sentence token trigger): `G.oopsLeft = freeOops(); G.typoCharged = 0;`
- On `startWave`: the same reset. On a zombie reaching the fence: the same reset (the kid gets a fresh start).
- `G.typoGrace -= dt` in `updateGame`.
- **Backspace (`dropTarget`) never costs HP.** Keys that are ignored today (Shift, digits in words mode, space in words mode) are still ignored, and they never count as typos.
- Accuracy (`G.keys`/`G.correct`) is counted exactly as it is today.
- A zombie reaching the fence: `damage(z.boss ? D.bossBite : D.bite)`. If `G.hp <= 0`, call `gameOver()`. This is the only path to game over.

Worst case on Easy with no upgrades: a kid typing randomly loses at most 1.5 HP between zaps and cannot go below 1 HP. On Hard every wrong key costs 1, but typos alone still can't end the run.

---

## 2. Currency: Coins

Name: **Coins** (icon: a gold coin with a small pumpkin stamp). Earned coins pile up in `G.runCoins` during the run. They are *banked* (multiplied, then saved) at stage clear, game over, quit-to-menu and `pagehide`.

| Event | Coins | Hook |
|---|---|---|
| Zap a zombie (letters/words) | 1, or 2 if `z.tier >= 2` | `typeChar` doom branch |
| Zap a sentence word | 1 | `typeSentenceChar` trig branch |
| Zap the sentence boss | +5 (on top of the 1) | `t.last` |
| Perfect sentence | +3 | `updateSentences` perfect branch |
| Streak milestone: every 10th zap in a row | +3 | `G.combo % 10 === 0` |
| Stage clear | `2 + stage` (3, 4, 5, ...) | `waveClear` first tick |
| Run end: score bonus | `floor(score / 250)` | `gameOver` |
| Run end: accuracy of 90% or more (and 20 or more keys) | +5 | `gameOver` |

**Multiplier at banking:** `mult = DIFFICULTY[G.diff].coinMult * (1 + 0.2 * up.magnet)` (×1 on Easy, ×1.25 on Medium, ×1.75 on Hard).
`banked = Math.round(G.runCoins * mult); save.coins += banked; G.runCoinsBanked += banked; G.runCoins = 0; saveGame();`

**Pop-ups:** each coin award shows a small gold `+1` next to the existing score pop (`scorePop`, second line, coin color).

**Expected income (tested by hand against `waveSize`/`planStage`):**
- First Words run on Easy, dies in stage 3: about 19 zaps (+22), 2 clears (+7), 1 streak (+3), score bonus (+3), for **about 35 coins**.
- Letters is similar (bigger waves, 1 coin each). Sentences pays a bit more (about 30 to 40 per stage, because of the boss).
- New save gift: **15 coins and 1 Juice Box** (so the power-up tray is taught on run 1).
- Result: a consumable after run 1, and a first permanent upgrade (50) after run 2. Full shop is about 2,800 coins, which is 60 to 80 runs of long-term goals.

---

## 3. Player level and XP (light)

- XP: +1 per zap, +5 per stage clear. Credited at banking time (not multiplied).
- XP to go from level L to L+1: `30 * L` (30, 60, 90, ...). Cap: level 20.
- Levels give a title (and the gold blaster at level 8). **No max-HP bonus per level** (removed: it made awkward totals like 1420).
- Titles (just for fun, shown on the menu): 1 Zapper Rookie, 3 Pumpkin Guard, 6 Zombie Buster, 10 Fence Hero, 15 Brain Saver, 20 Legendary Zapper.
- Level 8 unlocks the `gold` blaster color for free (see cosmetics).

---

## 4. Shop

### 4a. Permanent upgrades (`save.up[id]` = tier, 0 = not owned)
| id | Name | Icon | Kid text | Cost per tier | Effect in code |
|---|---|---|---|---|---|
| `maxhp` | Pumpkin Armor | 🛡️ | "More health! You can take more zombie bumps." | 50 / 150 / 300 | `maxHp += 20 * tier` |
| `oops` | Oops Shield | 🫧 | "More free oops after every zap." | 60 / 200 | `freeOops() = 1 + tier` |
| `magnet` | Coin Magnet | 🧲 | "Collect more coins every game!" | 80 / 200 / 400 | coin `mult *= 1 + 0.2 * tier` |
| `splash` | Splash Zapper | 💥 | "Big zaps push nearby zombies back!" | 100 / 250 | see below |
| `slow` | Sleepy Dust | 🌙 | "Zombies walk a little slower. Yawn!" | 120 / 300 | `zombieSpeed() *= 1 - 0.06 * tier`; also `G.sSpeed` |
| `regen` | Snack Pack | 🍎 | "Get a little health back with every zap." | 100 / 250 | on zap: `if (D.regen) heal(tier)` (on top of the perfect heal). Off on Hard |

**Splash, exactly:** in `hit(z, big)` when `big` is true (boss included), with `R = [0, 4, 6][tier]` and `P = [0, 2, 3.5][tier]`: for each other zombie `o` with state `walking` or `rising`, `!o.demo`, and an xz distance to `z` under `R`, set `o.root.position.z = Math.max(o.zStart, o.root.position.z - P)` and `o.flinch = 0.2`. Spawn a small cyan ring with `shockwave()` (it takes a color and scale parameter; defaults stay as today).

### 4b. Consumables (`save.items[id]`, max 3 of each, used from the tray)
Bought in the shop, carried between runs, and used up only when triggered.
| Slot | id | Name | Icon | Kid text | Cost | Effect |
|---|---|---|---|---|---|---|
| 1 | `juice` | Juice Box | 🧃 | "Slurp! Get 40 health back." | 15 | `heal(40)`. Disabled when `hp === maxHp` |
| 2 | `freeze` | Freeze Ray | ❄️ | "Freeze every zombie for 4 seconds!" | 20 | `G.freezeT = 4` (see below) |
| 3 | `bomb` | Pumpkin Bomb | 🎃 | "BOOM! Zap every zombie on the screen." | 30 | see below. Cooldown 8 s |

- **Freeze:** while `G.freezeT > 0`, walking speed is 0 (`sp = 0` in `updateZombies`, and the walk animation phase stops). The spawn timers do not tick (`G.spawnTimer` in `updateGame` and `updateSentences`). Typing still works and bolts still hit. The tags get a `.frozen` class. Using it again while active resets the timer to 4 (no stacking above 4).
- **Bomb in letters/words:** every non-demo zombie with state `walking` or `rising` is doomed, and each one gets `fire(z, true)` staggered by 0.06 s through `schedule`. Each one counts as a zap for `G.zapped`, coins and XP, with points of `10 * word.length` (no combo multiplier, and the combo is not increased). Clear `target`.
- **Bomb in sentences** (typing is the lesson, so there are no free kills): push every live token zombie back by 12 units (`z = max(zStart, z - 12)`), add a big shockwave, and no words are skipped.
- An item is disabled (dimmed, with a "nope" buzz) when: `G.state !== "playing"`, `G.clearTimer > 0`, the count is 0, the cooldown is active, there are no live zombies (freeze or bomb), or HP is full (juice).

### 4c. Cosmetics: blaster colors (`save.skins`, `save.skin`)
Recolors the blaster body (`gun` box `0xff9a3c`), the bolt color in `fire()` and the gem. One is equipped at a time.
| id | Name | Body / bolt | Cost |
|---|---|---|---|
| `pumpkin` | Classic Pumpkin | ff9a3c / ffe08a | free (owned) |
| `slime` | Slime Green | 9bdc6e / d8ffb0 | 20 |
| `galaxy` | Galaxy Purple | 9b5de5 / d6b8ff | 20 |
| `candy` | Candy Pink | ff7eb6 / ffd1e6 | 20 |
| `ice` | Ice Blue | 8fb8ff / e0f0ff | 20 |
| `gold` | Golden Zapper | ffc93c / fff1a8 | 100, or free at level 8 |

### 4d. Fun ammo (`save.ammos`, `save.ammo`)
What the blaster throws. One is equipped at a time; `fire()` clones a small 3D model (`AMMO_MODELS`) and flies it on its own arc and spin (`AMMO_FX`), and `ammoSplat()` adds its own chunks and sound on the hit. Game rules are the same for every ammo.
| id | Name | Cost | Flight |
|---|---|---|---|
| `zap` | Zap Bolts | free (owned) | the classic glowing bolt, in the blaster color |
| `arrow` | Arrows | 60 | fast, nearly straight, points where it flies |
| `banana` | Bananas | 80 | spins like a boomerang, "boing" |
| `pie` | Cream Pies | 100 | frisbee spin, cream splat |
| `brick` | Bricks | 120 | tumbles, dusty crash |
| `melon` | Watermelons | 150 | high lob, red and green splat |
| `barrel` | Barrels | 200 | rolls end over end, wooden bonk |
| `cat` | Flying Cats | 250 | superhero pose with a cape, meows, sparkle trail |
| `mix` | Surprise Mix | 300 | a random ammo (not zap) on every shot |

---

## 5. Hotkeys for consumables

The three slots sit in a fixed order, left to right: 1 Juice, 2 Freeze, 3 Bomb.

| Input | Letters / Words | Sentences |
|---|---|---|
| Digits `1` `2` `3` | Yes (digits are never typed in these modes) | **No** (digits are part of sentences) |
| Arrow keys `←` `↓` `→` | Yes | Yes |
| Tap/click the tray buttons | Yes | Yes |

- Why arrow keys: they have no text meaning and are never typed. `←` `↓` `→` sit in one physical row that matches the tray. Every keyboard has them. One rule works in all modes.
- Rejected: F1 to F3 (Mac needs `fn`, F1 opens help, F5 reloads), Tab (moves focus, and kids press it by accident), Enter (reserved for start/replay).
- `keydown`: handle these *before* the existing `e.key.length === 1` branch. Ignore `e.repeat`. Call `preventDefault()` (arrows scroll the page).
- The tray labels show the key: `1 / ←` in letters/words, and only `←` in sentences. On touch devices, no key labels are shown.
- **Mobile hidden input:** tray buttons must use `pointerdown` with `e.preventDefault()` (not `click`) so `#typeInput` keeps focus, then call `ui.input.focus()` as a backup. In the `input` handler, when the mode is not sentences, typed `1`/`2`/`3` also trigger slots.

---

## 6. Screens and flow

```
Menu ──Start──▶ Run ──HP 0──▶ Results ──Play again──▶ Run
 │  ▲                          │   │
 │  └────Change words──────────┘   └──Shop──▶ Shop ──Back──▶ Results
 └──Shop──▶ Shop ──Back──▶ Menu        (Pause ▸ Back to menu also banks coins)
```
The shop is reachable only from the menu and results screens, never mid-run.

**Menu (`#menu`):** at the top right of the panel, a coin pill (`🪙 135`) and a level chip (`Lv 4 Pumpkin Guard` with a thin XP bar). The actions row becomes `Start zapping` · `Shop` (ghost button) · hint. Update the how-to copy: *"Oops keys cost a tiny bit of health, but your first oops after each zap is free!"* and *"Use power-ups with ← ↓ → (or 1 2 3)."*

**Shop (`#shop`, a new `.overlay`, panel `max-width: 720px`):**
- Header: "Zapper Shop", with the coin pill on the right.
- Three sections, stacked (no tabs): **Power-ups**, **Upgrades**, **Blaster Colors**. Cards sit in a 2-column grid (1 column under 640px).
- Card contents: icon, name, kid text, tier pips (●●○), and a price button (`🪙 150`). States: affordable (orange button), too expensive (grey, "Need 20 more"), maxed ("MAX!" green chip), consumable full ("3/3"). Color cards show "Equip" or "Equipped".
- On buy: coin pill counts down, a `sfx.buy()` sound plays, the card bounces, and `saveGame()` runs immediately.
- Footer: `Back` button, and a small `Reset progress` text button (see section 7). Esc and Back return to the screen that opened the shop.
- Enter inside the shop activates the focused button only. The global Enter-to-start must check `G.state === "menu"`, and the shop sets `G.state = "shop"`.

**HUD (`#hud`):**
- Left: HP bar replaces `#lives`. It has a heart icon, a 180px bar (140px on mobile), and the text `72 / 100`. Fill colors: green above 60%, yellow above 34, red at 34 or below (pulse).
- Then Stage and Score as today, then a coin counter showing run coins before the multiplier (`🪙 23`).
- Power-up tray: a new `#tray`, fixed at bottom-left (`left: 16px`, above the safe area), with 3 round buttons (56px) showing icon, count badge and key label. Freeze shows a draining ring while active; bomb shows a cooldown ring. On mobile it sits bottom-left and `#tapType` stays centered. Set `pointer-events: auto` on the tray only.

**Results (`#over`):**
- Existing stats, plus a new **coins block**: the lines "Zaps +22 · Stages +7 · Streaks +3 · Score bonus +3", a "Difficulty/magnet bonus ×1.25" line when `mult > 1`, and a large **"+44 coins"** that counts up, with "Total: 🪙 179".
- XP bar that fills, with a "Level up! Level 4: Pumpkin Guard " chip.
- If an affordable item exists: the hint "You can buy something in the Shop!".
- Actions: `Play again` (focused, Enter) · `Shop` · `Change words`.

---

## 7. Persistence

A single key, **`zwz-save`**, read and written through the existing `store` helper (it already uses try/catch). Existing keys (`zwz-mode`, `zwz-helper`, `zwz-muted`) are unchanged; `zwz-fingers` (default true) toggles the finger hints. `zwz-diff` replaces `zwz-speed`, and the bests become `zwz-best-<mode>-<diff>` (see section 0).

```json
{
  "v": 1,
  "coins": 135,
  "lifetimeCoins": 410,
  "xp": 212,
  "up":    { "maxhp": 1, "oops": 0, "magnet": 1, "splash": 0, "slow": 0, "regen": 0 },
  "items": { "juice": 2, "freeze": 1, "bomb": 0 },
  "skins": ["pumpkin", "slime"],
  "skin": "slime",
  "ammos": ["zap", "banana"],
  "ammo": "banana",
  "stats": { "runs": 12, "zapped": 388 },
  "seenTrayTip": true
}
```
- `loadSave()`: `raw = store.get("zwz-save", null)`, deep-merged over `DEFAULT_SAVE` (15 coins, `items.juice = 1`, `skins: ["pumpkin"]`). Every number is clamped to an integer of 0 or more, and every tier is clamped to that item's max tier. Unknown ids are dropped. If the skin is not owned, fall back to `"pumpkin"`.
- `level` is derived from `xp` and never stored.
- Migrations: `if (raw.v < CURRENT) migrate(raw)`. A save with `v > CURRENT` is still loaded, reading only the known fields.
- `saveGame()`: `store.set("zwz-save", save)`. It is called on buy/equip, banking and item use.
- When storage is unavailable, the game runs with an in-memory save, and nothing breaks.
- **Reset progress:** a two-tap confirm. The first tap changes the label to "Tap again to erase coins & upgrades". It reverts after 3 s. The second tap removes `zwz-save` (only) and reloads `save = DEFAULT_SAVE`. Best scores are kept.

---

## 8. Balancing notes and edge cases

- **Game over:** only `damage()` from a zombie can reach 0. Typos are floored at 1 HP. Keep the `return` after `gameOver()` inside the `updateZombies` loop so that 2 zombies arriving in the same frame don't double-bank.
- **Banking once:** track `G.runCoins` (unbanked) plus `G.runCoinsBanked`. `bankCoins()` is idempotent when `runCoins === 0`. Call it in `waveClear` (first tick), `gameOver`, `toMenu` (quit from pause), and `window.addEventListener("pagehide", ...)` when a run is in progress. The results screen shows `G.runCoinsBanked`.
- **Difficulty fairness:** Hard pays ×1.75 coins but has no healing and charges for every typo. Easy pays ×1 with 0.5 HP typos and big heals. Kids pick their own risk.
- **Upgrades vs. best score:** best scores stay per mode and are not reset by upgrades. That is fine for this audience.
- **Letters mode:** each zap resets the free oops, so a 6-year-old gets 1 free wrong key per letter. On Easy they get 2 free wrong keys per letter, and charged damage is 0.5.
- **Sentence mode:** the typo rules are the same (zap = token trigger). "Perfect" for a token means `!t.z.oops`. The bomb only knocks back. Freeze is the best power-up here, and it pauses `G.spawnTimer` for the token queue.
- **Freeze plus fence:** a frozen zombie cannot reach the fence. Zombies in the `rising` state finish rising, then stand frozen.
- **Bomb during `rising`:** included. Don't count `doomed` zombies twice. The bomb must not fire at demo zombies.
- **Stage clear with items:** the tray is disabled during `clearTimer` and banners.
- **Pause/blur:** item keys are ignored when `G.state !== "playing"`. `freezeT` and cooldowns tick on `gdt`, so they pause correctly.
- **Key autorepeat:** ignore `e.repeat` for items. Typing is unchanged.
- **Multiple tabs:** last write wins, which is acceptable.
- **Tuning dials, if runs feel too easy after upgrades:** raise the bite by stage (`D.bite + 2 * (stage - 1)`, capped at `D.bossBite`) rather than cutting heals.

---

## 9. Developer checklist (priority order)

**P0: MVP (playable loop)**
1. `RPG` constants, `DEFAULT_SAVE`, `loadSave()`, `saveGame()`, `bankCoins()`, and `level()`/`maxHp()`/`freeOops()` helpers.
2. Replace hearts with HP: `G.hp`/`G.maxHp`, `damage()`, `heal()`, and `renderHud()` drawing the bar. Change `waveClear` to `heal(D.healStage)` with the copy "Health restored!" (on Hard, where the heal is 0, show "Great zapping!"). The fence hit uses `damage`.
3. Typo damage in `miss()` through `typoHurt()`, plus the free-oops/grace/cap/floor resets in the zap paths and `startWave`.
4. Coin awards (section 2 table), the HUD coin counter, and banking at clear/over/quit/pagehide.
5. The results-screen coins block, plus the menu coin pill.
6. Shop overlay with `maxhp`, `oops`, `magnet` and the 3 consumables. Buy logic and saving.
7. The `#tray` with 3 slots, keys (`1-3` in letters/words, arrows in all modes), `pointerdown` for touch, and effects for juice/freeze/bomb.

**P1: depth**
8. `splash`, `slow`, `regen` upgrades.
9. XP/levels, titles and the level-up chip.
10. Blaster color cosmetics (material swap in `fire()` and on the gun).
11. Hit FX: the red vignette, the HP bar shake, floating `-34`/`-2`, and the "Oops shield!" bubble.

**P2: polish**
12. SFX: `coin()` (a quick two-note ding), `buy()`, `levelUp()`, `heartbeat()` at low HP, `freeze()`, and `bomb()` (reuse `boom`).
13. Reset progress, the first-run tray tip ("Press ← for a Juice Box!", shown once, `seenTrayTip`), and keyboard focus order in the shop.
14. Update the how-to copy, and add `prefers-reduced-motion` fallbacks for all new animations.

---

## 10. Asks for the graphics artist

1. **HP bar:** a chunky rounded bar in the Lilita/Creepster style with a cream outline. The fill is slime green to pumpkin yellow to heart red. There is a heart cap on the left, and a "ghost" trail segment that drains 0.4 s after a hit (the classic fighting-game lag bar). Low HP gives a red pulse glow.
2. **Coin icon:** an inline SVG, a gold coin with an embossed pumpkin, readable at 18px and 32px. Also a coin-count "tick" animation.
3. **Shop cards:** a card component with an icon well (64px emoji or SVG), tier pips, a price button in affordable/locked/max/equipped states, and a purchase bounce plus sparkle burst. It must work in a 2-column and 1-column layout.
4. **Player-hit effect:** a zombie hit gives a red radial vignette (a new `#hurt` element, like `#flash`) plus the existing camera shake. A typo hit gives only a tiny bar flicker and a floating "-2" (no shake, no vignette, to keep it gentle). "Oops shield!" is a soap-bubble pop at the bar.
5. **Power-up VFX:**
   - Juice: green plus signs rising from the bottom of the screen.
   - Freeze: a blue frosty screen edge, zombies tinted icy with an emissive override and ice crystal particles, and tags with a frost border (`.tag.frozen`).
   - Bomb: a pumpkin projectile arcs from the blaster, then a big orange shockwave and confetti (reuse `bossBlast` pieces).
   - Splash: a small cyan ring.
6. **Tray buttons:** round 56px buttons with an icon, a count badge, a key hint label, a radial cooldown/duration overlay and a dimmed disabled state.
7. **Blaster model upgrades:** the 6 color palettes, plus visible upgrade bits on the gun. Pumpkin Armor gives a small metal plate, Coin Magnet a tiny horseshoe magnet on top, and Splash Zapper a wider barrel tip. Keep the box/cylinder low-poly style.
8. **Results screen:** a coin count-up, an XP bar fill, and a "Level up!" badge in slime green like `.newbest`.
9. **Level chip and title** on the menu, in the same pill style as `.newbest`.
