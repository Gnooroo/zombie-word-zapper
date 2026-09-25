# Zombie Word Zapper

A 3D typing game for kids (three.js). Type the words floating over the zombies to zap them before they reach the pumpkin patch.

- **Levels:** Letters, Words, Sentences (with punctuation, numbers and symbols that grow by stage; the last word of each sentence is a boss).
- **Difficulty:** Easy / Medium / Hard (zombie speed, typo forgiveness, bite damage, healing, coin bonus).
- **RPG layer:** health bar, coins saved in the browser, a shop with power-ups (Juice Box, Freeze Ray, Pumpkin Bomb), permanent upgrades, blaster colors, and player levels.

## Play

It's a single static file. Open `index.html` in a browser, or serve the folder:

```bash
python3 -m http.server 8765
```

Progress is saved in `localStorage` (`zwz-save`, `zwz-diff`, `zwz-best-*`).

## Files

- `index.html` – the whole game (HTML, CSS, JS; three.js r128 from cdnjs).
- `RPG_DESIGN.md` – the design spec for the RPG systems and difficulty presets.
