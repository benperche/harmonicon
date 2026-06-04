# Harmonicon — developer reference

A single-page, dependency-free harmony reference (Roman numerals + figured bass).
**Everything lives in `index.html`** — HTML, CSS, and JS in one file. No build step;
open the file or serve the folder (`python3 -m http.server`).

## Layout of `index.html`

- **`<style>`** (top): all CSS. Note-display classes worth knowing:
  - `.td-notes` — chord-tones cell/row. Uses `word-spacing` so adjacent bare-letter
    notes (e.g. `F A`) don't visually collide into `FA`.
  - `.note-seventh` — the *optional* 7th label, rendered as `(7th: X)` in italics.
    Resets `word-spacing: normal` so the label text stays tight.
  - `.bass-hl` — underlined bass note of an inversion.
- **JS spelling helpers** (~line 615+): `SH`/`FL` tables, `nn(pc, sharp)`,
  `buildDiatonicSpelling`, `spellByLetter`, `LETTERS`/`LETTER_PC`.
- **`getDiatonicChords(rootPc, mode, useSharp)`** (~line 819): the core builder.
  Returns an array of chord objects. Sub-sections inside it: diatonic, secondary
  dominants, **augmented 6ths**, common-tone, Neapolitan, borrowed.
- **Render functions** (~line 1400+): `renderChords` (card view),
  `renderChordsFunctional` (grouped-by-function tiles), plus bass-lookup renderers.
  These iterate `ch.inversions[].allNotes` to print chord tones.
- **State** `S` (~line 1224): `S.keyIndex`, `S.mode`, etc. `COF_MAJ`/`COF_MIN` map
  circle-of-fifths position → `{name, pc, sharps}`.

## Note-spelling conventions (important)

Pitch classes are integers 0–11. Two ways to turn a pc into a note name:

1. **`nn(pc, sharp)`** — generic: picks from the sharp or flat table by the key's
   preference. Fine for diatonic notes, **wrong for chords whose members have a
   fixed scale-degree identity**, because it ignores letter names.
2. **`spellByLetter(letterIdx, pc)`** — spells `pc` on a *specific letter* (0=C..6=B,
   taken mod 7). The accidental is derived from the gap between `pc` and that
   letter's natural pc, so it can yield single **and double** accidentals. This is
   how scale-degree-locked chromatic chords must be spelled.

`rli` inside `getDiatonicChords` is the tonic's letter index. "The Nth letter above
the tonic" is `rli + (N-1)`.

## Augmented 6th chords (the strict rules)

All three types: **♭6̂ in the bass, ♯4̂ on top** — the interval between them is an
augmented 6th, which *is* the chord's identity. So both notes must keep their
scale-degree spelling regardless of key.

- `♭6̂` = `spellByLetter(rli + 5, rootPc + 8)` — 6th letter, lowered.
  E♭ major → **C♭**, never B. (This was the bug: `nn` gave B.)
- `♯4̂` = `spellByLetter(rli + 3, rootPc + 6)` — 4th letter, raised.
- `1̂` = `spellByLetter(rli, rootPc)`.
- French adds `2̂` = `spellByLetter(rli + 1, rootPc + 2)`.
- German adds `♭3̂` = `spellByLetter(rli + 2, rootPc + 3)`.

The ♭6̂→♯4̂ pair is always **six letter-names apart** (a written aug 6th). Remote
keys legitimately produce double accidentals (e.g. A♭ major German = F♭–A♭–C♭–D).

**Essential 7th, not optional:** the French `2̂` and German `♭3̂` are obligatory
chord members, unlike the 7th of a V⁷. They carry `essential7: true` on both the
chord object and its inversion, and the render functions check `!inv.essential7`
(and `ch.essential7`) to render them as **plain chord tones** — *not* wrapped in the
`(7th: …)` brackets used for V⁷ etc.

## Other scale-degree-locked chords

The **Neapolitan (N⁶)** is spelled the same strict way: a major triad on ♭2̂, with
`♭2̂ = spellByLetter(rli+1, ...)`, `4̂ = spellByLetter(rli+3, ...)`,
`♭6̂ = spellByLetter(rli+5, ...)`. E♭ major → `F♭ – A♭ – C♭`. Remote flat keys
yield correct double flats (A♭ major → `B𝄫 – D♭ – F♭`).

## Verifying changes

Serve and drive the page; the builder is callable directly:

```js
getDiatonicChords(3, 'major', false)        // E♭ major
  .filter(c => c.section === 'aug6')
  .map(c => `${c.rn}: ${c.chordNotes.join(' ')}`)
```

When the static server caches an old copy, hard-navigate with a cache-buster:
`window.location.href = '/index.html?v=' + Date.now()`.
