# Zombie Word Zapper

A 3D typing game for kids (three.js). Type the words floating over the zombies to zap them before they reach the pumpkin patch.

- **Levels:** ABC (small letters only, starting with f and j and adding a few more each stage), Letters (small letters in stages 1-2, capitals with Shift in 3-4, a mix in 5-6, then numbers from stage 7, symbols from 9, Shift symbols from 11, and every other key from 13 and 15), Words (same schedule: Capitalized words from stage 3, some ALL CAPS from 5, then words with numbers and symbols like cat7, don't, #1, [box] and ^_^), Sentences (punctuation, numbers and symbols that grow every 2 stages until every key is used by stage 13; the last word of each sentence is a boss).
- **Difficulty:** Frozen (zombies never move and nothing hurts, so it's pure practice until you quit; half coins) / Easy / Medium / Hard (zombie speed and count, typo damage and forgiveness, blaster jams, bite damage, healing, coin bonus).
- **Finger hints:** the keyboard helper color-codes every key by finger, and two big hands on either side of the zombie lane light up the finger to use, with the key in a bubble at its fingertip (plus the other pinky for Shift). They follow the next key, or the closest zombie's first letter.
- **Stage map:** every level and difficulty remembers the stages you've cleared. Start from any unlocked stage, and earn up to 3 stars on each (clear it, 90% right, no bites). After a run, Play again carries on from the stage you reached.
- **Tricky keys and speed:** the results show your typing speed (words a minute, or keys a minute in ABC and Letters) with your best, and the keys you missed most, colored by finger. **Practice these** starts a Frozen run with just those keys, and in normal runs about a quarter of new zombies bring your tricky keys back. Quitting a Frozen run from Pause shows these results too.
- **RPG layer:** health bar, coins saved in the browser, a shop with power-ups (Juice Box, Freeze Ray, Pumpkin Bomb), permanent upgrades, blaster colors, fun ammo (arrows, bananas, cream pies, bricks, watermelons, barrels, flying cats, or a surprise mix), and player levels.

## Play

**Play online:** https://gnooroo.github.io/zombie-word-zapper/

Or run it locally. It's a single static file. Open `index.html` in a browser, or serve the folder:

```bash
python3 -m http.server 8765
```

Progress is saved in `localStorage` (`zwz-save`, `zwz-diff`, `zwz-best-*`).

## Files

- `index.html` – the whole game (HTML, CSS, JS; three.js r128 from cdnjs).
- `RPG_DESIGN.md` – the design spec for the RPG systems and difficulty presets.
