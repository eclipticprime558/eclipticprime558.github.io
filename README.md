# NSRT → MRT Converter

Converts **Northern Sky Raid Tools** cooldown plan exports into **Method Raid Tools** note format.

Paste your NSRT export, click Convert, paste the result into MRT.

**[Open the tool →](https://eclipticprime558.github.io/nsrt-mrt/)**

---

## Usage

1. In Wowutils/Viserio, export your cooldown plan (Copy All Notes)
2. Paste the NSRT output into the left panel
3. Adjust options if needed (difficulty override, bossAbility tokens, name order)
4. Click **Convert** or press `Ctrl+Enter`
5. Click **Copy** and paste into MRT's note editor (`/mrt note`)

## Format mapping

| NSRT | MRT |
|---|---|
| `EncounterID:X;Difficulty:Heroic;Name:Y` | `EncounterID:X Difficulty:15 Name:Y` |
| `time:X` | `{time:X}` |
| `spellid:X` | `{spell:X}` |
| `bossSpell:X` | `{bossAbility:X}` |
| `tag:PlayerName` | `PlayerName` (plain text) |
| `ph:X` | *(dropped)* |

### Difficulty numeric values

| Difficulty | Value |
|---|---|
| LFR | 17 |
| Normal | 14 |
| Heroic | 15 |
| Mythic | 16 |

## No install required

Single HTML file. Open in any browser. No server, no build step, no dependencies.

## License

MIT
