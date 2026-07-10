# Harmonicon — Roman Numerals and Figured Bass Harmony Reference

> *Vibecoded using [Claude](https://claude.ai) by Anthropic.*

A single-page web app for labelling harmonies in tonal classical works using Roman numerals and figured bass notation. Built to run entirely in the browser with no dependencies — just open `index.html`.

**Live demo:** `https://benperche.github.io/harmonicon/`

## Features

### Chord reference
- **Circle of fifths key selector** — click (or keyboard-navigate) any segment to set the key; toggle major/minor; enharmonic keys on the outer ring. Your key choice is remembered for a day.
- **Diatonic chords** — every chord of the key with the optional 7th shown in brackets
- **Figured bass inversions** — every inversion with its Roman numeral suffix (⁶, ⁶₄, ⁷, ⁶₅, ⁴₃, ⁴₂, °⁷ …), bass note underlined
- **Cadential 6/4** — V⁶₄ (the tonic triad over the dominant), grouped with the dominant function
- **Secondary dominants** — V⁷/x of each applicable degree
- **Secondary leading-tone chords** — vii°⁷/x of the same degrees, with full °⁶₅/°⁴₃/°⁴₂ inversions
- **Augmented sixth chords** — It⁺⁶, Fr⁴₃, Ger⁶₅ with strict scale-degree spelling (♭6̂ bass, ♯4̂ top — including correct double accidentals in remote keys)
- **Neapolitan (N⁶)** and **borrowed chords** (modal mixture) — also strictly spelled
- **Common tone chords** — CT°⁷ and CT V⁷ with the sustained tone indicated
- **By Function view** — the same chords grouped Tonic / Pre-dominant / Dominant

### Sound & piano
- **Audio playback** — dependency-free Web Audio piano voice; ▶ strums, ∿ sustains. A Playback toggle plays triads only (default) or with the 7th. Plays through the media channel on iOS, so it works with the silent switch on.
- **Piano keyboard** — a collapsible keyboard along the bottom shows every chord you hover, expand, or play, colour-coded by role (root / 3rd / 5th / 7th) with a matching legend. Click any key to hear it.
- **Orienting progression** — one click plays I–IV–V–I (i–iv–V–i in minor) to settle your ear in the key.

### Lookup & analysis
- **Chord lookup** — pick pitch classes, find every matching chord in the key with all applicable inversions
- **Identify from bass note** — choose the bass you see; get every chord/inversion it could be
- **Modes tab** — the seven church modes on any root, with altered degrees shaded

### App
- **Installable / offline** — a small service worker makes it a PWA; works with no connection once visited
- **Linkable tabs** — `#lookup`, `#bass`, `#modes` open that tab directly
- **Responsive** — desktop, tablet, and phone, portrait or landscape

## Local use

No build step. Open `index.html` directly, or serve the folder (`python3 -m http.server`) for service-worker support.

## Figured bass quick reference

| Chord type | Root | 1st inv | 2nd inv | 3rd inv |
|---|---|---|---|---|
| Triad | (plain) | ⁶ | ⁶₄ | — |
| 7th chord | ⁷ | ⁶₅ | ⁴₃ | ⁴₂ |
| Diminished 7th | °⁷ | °⁶₅ | °⁴₃ | °⁴₂ |

The bass note in each inversion is underlined. An optional 7th is shown in *(brackets)*; essential chord members (e.g. the French ⁴₃'s 2̂) are shown as plain tones.
