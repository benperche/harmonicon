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
  Returns an array of chord objects. Sub-sections inside it: diatonic,
  **cadential 6/4**, secondary dominants, **secondary leading-tone** (vii°⁷/x,
  a dim7 on the semitone below the target root, spelled with `spellStack` on
  the letter below the target letter), **augmented 6ths**, common-tone,
  Neapolitan, borrowed. `rnWithSuffix` collapses the double ° when a dim rn
  (vii°, vii°/x) meets a °-prefixed figured-bass suffix (vii°⁷, not vii°°⁷).
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
   how scale-degree-locked chromatic chords must be spelled. `buildDiatonicSpelling`
   is a thin wrapper that calls it for each scale degree.
3. **`spellStack(rootLetterIdx, pcArr)`** — spells a root-position tertian
   (stacked-thirds) chord: members advance by a third, so letters advance by two
   each (C–E–G–B). Use for any built-by-interval chord whose members may be
   chromatic to the key (CT°7, CT V7, borrowed major triads). Passing the chord's
   own root letter keeps every member correctly spelled.

`rli` inside `getDiatonicChords` is the tonic's letter index. "The Nth letter above
the tonic" is `rli + (N-1)`. The CT and borrowed chords pass `rli + <degree>` as the
chord-root letter to `spellStack`.

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

## Audio playback

Dependency-free Web Audio (section "AUDIO PLAYBACK" in `index.html`):

- **Voice:** a cached piano-ish `PeriodicWave` (additive harmonics) per note, a
  per-note lowpass brightness envelope, and a fast-attack amplitude envelope.
  Everything runs through a shared gain→compressor master chain (`masterOut`)
  that limits sustained chords so they don't clip.
- **`playMidis(midis, mode)`** — `'strum'` arpeggiates (note roll, ~1.4s decay);
  `'sustain'` strikes all notes together and holds ~2s before release.
- **`chordVoicing(noteNames, bassName)`** places the bass lowest (~C3) and stacks
  the rest above, so inversions sound inverted. `noteNameToPc` parses the spelled
  names (incl. 𝄪 𝄫) back to pitch classes.
- **iOS routing:** `audioCtx()` sets `navigator.audioSession.type = 'playback'`
  so sound plays through the media channel (audible with the mute switch on),
  not the ringer. Guarded in try/catch — it's a Safari-only API.
- **UI:** `playControlsHTML` emits the strum / sustain pair as **inline SVG**
  icons (a triangle and a sine wave), not text glyphs — iOS gives `▶` an emoji
  presentation, so SVG keeps them identical across platforms. A single
  capture-phase delegated `click` listener handles every `.play-btn` and calls
  `stopPropagation` so playing doesn't toggle the card/tile open.
- **Figure-driven 7ths (no global toggle):** every button plays exactly its
  `data-notes`. A card/tile header plays the plain triad and its optional 7th
  is rendered as a **playable `(7th: X)` chip** (`seventhChipHTML`, class
  `.seventh-chip`, handled by the same delegated listener and hover preview)
  that plays the chord with the 7th. Inversion rows always play what their
  figure shows (a row labelled V⁶₅ includes the 7th). Chords whose rn itself
  carries ⁷ (CT°⁷, CT V⁷) play all four from the header and get no chip;
  essential-7th chords (aug6) are untouched. `data-opt7` now only colours the
  7th blue on the keyboard — it never strips notes.

## Cadential 6/4

The `cad64` section is a single fixed chord: the tonic triad in second inversion
over 5̂ (dominant in the bass), labelled **V⁶₄** (literally I⁶₄, but it prolongs
the dominant — `getChordFunction` returns `'dominant'`). `triadNotes` is kept in
root position (`[1̂, 3̂, 5̂]`) so `chordRoles` colours the piano correctly (root =
1̂, 5th = 5̂); `chordNotes`/`allNotes` are bass-first (`[5̂, 1̂, 3̂]`) for the
voicing and the underlined bass.

## Piano keyboard

Fixed bottom bar (`#piano-bar`), section "PIANO KEYBOARD" in `index.html`:

- `buildPiano()` lays out keys for MIDI `PIANO_LO..PIANO_HI` (C3–E6, wide enough
  for every voicing): white keys flow, black keys absolutely positioned over the
  gaps. Each key carries `data-midi`. Called once in init.
- Clicking a key plays that single note (`playMidis([m])`) and flashes it.
- `highlightChord(names, bassName, label, roles)` colour-codes keys by
  scale-degree **role** (`roles = {root, fifth, seventh}` note names, from the
  button's `data-root`/`data-fifth`/`data-opt7`): root = red, 3rd/other = teal,
  5th = teal-blue, 7th = blue. Each role lights in every octave, but nothing
  **below the bass** (`bassName`, the floor) is shown — so a root-position chord
  starts at its root and an inversion at its bass. `chordRoles(ch)` supplies the
  root/5th (aug6 has no tonal root, so it anchors on its lowest note, no 5th).
  The `#piano-legend` swatches mirror these four colours. RN + notes show in
  `#piano-label`.
- Triggers (all via `highlightFromHost`, which reads the host's `.play-btn`
  data attributes so the Triad-only toggle is respected):
  - **Playing** a chord (the click handler also calls `highlightChord`).
  - **Hovering** a card/tile or an inversion row — a delegated `mouseover` on
    `#chord-grid`; an inversion row highlights that inversion's bass.
  - **Tapping to expand** a card/tile (the touch fallback for hover).
- `togglePiano()` collapses the bar (and shrinks `body` padding via
  `body.piano-collapsed`). On narrow screens `#piano-scroll` scrolls horizontally.
- iOS long-press selection is suppressed with `-webkit-user-select: none` +
  `-webkit-touch-callout: none` on `#piano-bar *` (the unprefixed `user-select`
  alone doesn't stop Safari's selection/magnifier).

## App shell (persistence, tabs, PWA, a11y)

- **Key persistence:** `saveKeyState()` (called from `renderChords`) stores
  `{keyIndex, mode, enhKey, ts}` in localStorage under `harmonicon-key`;
  `restoreKeyState()` discards it after 24h (TTL check on read — localStorage
  has no native expiry). Deliberate: remembered for a day, not forever.
- **Tab deep links:** `#lookup` / `#bass` / `#modes` open that tab on load;
  `showTab` mirrors the current tab into the hash via `history.replaceState`
  (`chords` = clean URL).
- **PWA:** `sw.js` — network-first for the document, cache-first for static
  assets, same-origin only. Registered at the end of init. Bump its `CACHE`
  name when changing cached-asset strategy.
- **COF a11y:** `makeCofFocusable` gives every circle-of-fifths segment
  `tabindex/role/aria-label` + Enter/Space activation; focus ring styled via
  `.cof-hit:focus-visible path`.
- **Orienting progression:** `playProgression()` (button under the key
  display) plays I–IV–V–I / i–iv–V–i, one chord per ~0.9s, highlighting each
  on the piano; the final chord becomes `_lastPlayedHL`.
- **Sticky piano highlight:** `_lastPlayedHL` tracks the last *played* chord;
  `mouseleave` on `#chord-grid` reverts the hover preview to it.

## Verifying changes

Serve and drive the page; the builder is callable directly:

```js
getDiatonicChords(3, 'major', false)        // E♭ major
  .filter(c => c.section === 'aug6')
  .map(c => `${c.rn}: ${c.chordNotes.join(' ')}`)
```

When the static server caches an old copy, hard-navigate with a cache-buster:
`window.location.href = '/index.html?v=' + Date.now()`.
