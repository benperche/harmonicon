# Harmonicon — Roman Numerals and Figured Bass Harmony Reference

> *Vibecoded using [Claude](https://claude.ai) by Anthropic.*

A single-page web app for labelling harmonies in tonal classical works using Roman numerals and figured bass notation. Built to run entirely in the browser with no dependencies — just open `index.html`.

**Live demo:** `https://benperche.github.io/harmonicon/`

## Features

- **Circle of fifths key selector** — click any segment to set the key; toggle between major and minor
- **Diatonic chord reference** — all diatonic chords for the selected key, with the diatonic 7th shown in brackets as an optional 4th note
- **Figured bass inversions** — every inversion listed with its Roman numeral suffix (⁶, ⁶₄, ⁷, ⁶₅, ⁴₃, ⁴₂, °⁷ etc.), bass note highlighted
- **Secondary dominants** — V and V⁷ of each applicable diatonic chord
- **Diminished 7th chords** — vii°⁷ with full inversion table (°⁷, °⁶₅, °⁴₃, °⁴₂)
- **Augmented sixth chords** — It⁺⁶, Fr⁴₃, Ger⁶₅ with correct enharmonic spelling
- **Common tone chords** — CT°⁷ and CT V⁷ with common tone indicated
- **Chord lookup** — select any combination of pitch classes and find every diatonic or secondary dominant match in the chosen key, with all applicable inversions
- **Enharmonic spelling toggle** — switch between auto (key-appropriate), sharps, and flats
- **Responsive layout** — works in portrait and landscape on desktop, tablet, and mobile

## Local use

No build step required. Just open `index.html` directly in any modern browser.

## Figured bass quick reference

| Chord type | Root | 1st inv | 2nd inv | 3rd inv |
|---|---|---|---|---|
| Triad | (plain) | ⁶ | ⁶₄ | — |
| 7th chord | ⁷ | ⁶₅ | ⁴₃ | ⁴₂ |
| Diminished 7th | °⁷ | °⁶₅ | °⁴₃ | °⁴₂ |

The bass note in each inversion is underlined. The diatonic 7th is shown in *(brackets)* alongside each inversion.