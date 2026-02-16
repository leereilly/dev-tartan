# Dev Tartan

**Every developer deserves their own clan.**

Dev Tartan generates a unique, deterministic tartan pattern from any GitHub username. Same username, same tartan, every time. It's like a coat of arms, but for people who think `vim` vs `emacs` is a hill worth dying on.

Built using [GitHub Copilot CLI](https://github.com/features/copilot/cli) for the [GitHub Copilot CLI Challenge](https://dev.to/leereilly/dev-tartan-deterministic-plaid-from-a-github-username-101m).

👉 **[Try it live](https://leereilly.net/dev-tartan)**

![Dev Tartan](demo.webp)

## How It Works

Your GitHub username goes in. A mass of intersecting colored rectangles comes out. In between? _Math_.

### Step 1: Hashing - Your Username Becomes Entropy

Your username is lowercased and fed into the [SHA-256](https://en.wikipedia.org/wiki/SHA-256) cryptographic hash function via the Web Crypto API. This produces 32 bytes (256 bits) of deterministic, uniformly distributed, avalanche-effected goodness. Change one letter in your username and the entire tartan changes. `octocat` and `Octocat`? Same tartan, since usernames are lowercased before hashing. But `octocat` and `octocot`? Completely different plaids. That's the [avalanche effect](https://en.wikipedia.org/wiki/Avalanche_effect), baby.

```
"octocat" → SHA-256 → [0x9b, 0x2a, 0xf1, 0x73, ... 28 more bytes of chaos]
```

### Step 2: Sett Derivation - From Bytes to Stripes

A tartan's repeating pattern is called a **sett**. Dev Tartan derives one from the hash bytes like so:

| Hash Byte(s) | Purpose |
|---|---|
| `byte[0]` | Number of colors: `4 + (byte % 3)` → 4, 5, or 6 colors |
| `bytes[1..6]` | Color selection: each byte picks from a 20-color palette of traditional Scottish dye colors (Dark Navy, Hunter Green, Crimson, Gold, etc.) via modular arithmetic |
| `bytes[8..13]` | Thread counts (stripe widths), mapped into three buckets: **thin** (2–6), **medium** (8–16), or **wide** (18–36) based on byte value thresholds |

Duplicate colors are avoided by bumping the palette index by 7 (a prime-ish offset, because even tartans deserve a little number theory).

### Step 3: Symmetry - The Mirror Trick

Real tartans are **symmetric**: the sett plays forward and then reflects back, like a palindrome. Given stripes `[A, B, C, D]`, the full sequence becomes:

```
A  B  C  D  C  B  A  B  C  D  C  B  A  ...
└──forward──┘  └─reverse─┘  └──forward──┘
```

This is called **reflective symmetry** in tartan weaving. It's what gives tartans that satisfying, kaleidoscopic quality. The code does this by concatenating the sett with a reversed copy (minus the first and last elements, to avoid doubled stripes at the pivot points). If you've ever reversed a linked list in a job interview, congratulations - you were unknowingly training to weave tartan.

### Step 4: Rendering - Warp, Weft, and SVG Rectangles

Real tartan is a **2/2 twill weave**: warp threads (vertical) and weft threads (horizontal) interlace at right angles. We fake this with layered SVG rectangles:

1. **Background fill** - the first color of the sett floods the canvas
2. **Warp stripes** (vertical `<rect>` elements) at 90% opacity
3. **Weft stripes** (horizontal `<rect>` elements) at 55% opacity with `mix-blend-mode: multiply` - this simulates thread crossings where colors optically blend, just like real woven cloth
4. **Twill texture overlay** - a tiny 4×4px repeating pattern of alternating semi-transparent light/dark squares, mimicking the diagonal twill ridges visible in real tartan fabric

The sett is scaled so roughly **3 full repeats** tile across the 500×500 SVG canvas.

### Step 5: Ambiance - Background Tartanification

After rendering, a miniature version of your tartan is generated as a `data:` URI SVG tile (160×160px) with extremely low opacity (6–8%) and set as the page's repeating background. Because why stop at _one_ tartan when you can tartanify the entire viewport?

## The Palette

Twenty colors inspired by historical Scottish dye sources, from woad-adjacent navies to lichen-derived golds:

| Color | Hex | | Color | Hex |
|---|---|---|---|---|
| Dark Navy | `#1c2541` | | Crimson | `#8b1a1a` |
| Royal Blue | `#2b4c7e` | | Scarlet | `#c41e3a` |
| Azure | `#3a7ca5` | | Rust | `#a0522d` |
| Sky Blue | `#6fb3d2` | | Burnt Orange | `#c76f30` |
| Hunter Green | `#1a472a` | | Gold | `#c5a041` |
| Forest Green | `#2d5a27` | | Straw | `#d4b94e` |
| Moss Green | `#4a7c59` | | Purple | `#5b2c6f` |
| Sage | `#87a96b` | | Heather | `#7d5a9e` |
| Black | `#1a1a1a` | | Charcoal | `#3d3d3d` |
| Ivory | `#f0e8d0` | | Grey | `#7a7a7a` |

## Features

- 🔒 **Deterministic** - same username always produces the same tartan (it's a pure function of SHA-256)
- 🎨 **Authentic-ish** - symmetric sett, twill texture, traditional color palette
- 📥 **Downloadable** - export your tartan as SVG
- 🔗 **Shareable** - URL query parameters (`?username=octocat`) and social sharing links
- 🪶 **Zero dependencies** - one HTML file, no build step, no npm, no node_modules, no existential dread
- 🎵 **No bagpipes** - we considered auto-playing _Scotland the Brave_ on page load, but decided some traditions are best left in the Highlands

## Running Locally

```bash
open index.html
```

That's it. It's a single HTML file. Your build pipeline is your browser.

## License

Do whatever you want with it. Put it on a kilt. Print it on a mug. Use it as your desktop wallpaper. Make it your GitHub avatar. Tattoo it on your forearm. We're not your parents.

