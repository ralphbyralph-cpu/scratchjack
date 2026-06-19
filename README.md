# Handoff: ScratchJack Mobile App (v1)

## Overview
ScratchJack is a scratch-off lottery odds tracker. The app ranks every state lottery's
scratch-off tickets by expected value (EV) using live "prizes remaining" data, surfaces
the channel's YouTube videos, explains the methodology, and wraps the whole thing in a
gamified, Vegas-neon casino theme (daily streaks, coins, levels, badges, and a
scratch-to-reveal "Top Bet").

This handoff covers **v1 = the entire design**: a login/sign-up gate plus five tabs —
**Rankings, Videos, Guide, Research, Rewards**.

**Target stack: React Native + Expo** (managed workflow). The reasoning, exact tokens,
copy, and interactions below are written so a developer who was not in the original
session can build the whole app from this README alone.

---

## About the Design Files
The files in `design/` are a **design reference**, not production code to ship.
`ScratchJack App.dc.html` is an HTML/JS prototype that demonstrates the intended look,
layout, copy, and interactions. `ios-frame.jsx` is only a preview device bezel (status
bar / dynamic island / home indicator) used to frame the prototype in the browser — **do
not port it**; Expo runs on the real device chrome.

Your task is to **recreate these designs natively in React Native / Expo** using its
ecosystem (React Navigation, Reanimated, etc.), not to embed the HTML. Treat the HTML as
the source of truth for visuals and behavior.

## Fidelity
**High-fidelity.** Colors, typography, spacing, copy, and interactions are final. Match
them pixel-for-pixel (translating CSS px → RN density-independent units 1:1). Where the
prototype uses web-only tricks (CSS `backdrop-filter`, canvas scratch), equivalents are
specified below.

---

## Recommended Expo Architecture

```
app/
  _layout.tsx              # root: auth gate -> (tabs) or /login
  login.tsx                # login / sign-up screen
  (tabs)/
    _layout.tsx            # bottom tab navigator (5 tabs)
    rankings.tsx
    videos.tsx
    guide.tsx
    research.tsx
    rewards.tsx
components/
  Background.tsx           # shared felt-gradient backdrop
  AppHeader.tsx            # logo + coins + streak chips
  ScratchCard.tsx          # canvas/skia scratch-to-reveal
  TicketRow.tsx
  VideoCard.tsx
  ...
theme/tokens.ts            # all design tokens below
state/usePlayer.ts         # coins, streak, level, claimed, scratched
assets/
  scratchjack-crest.webp
```

**Key libraries**
- **expo-router** (or React Navigation) — stack for auth + bottom tabs for the app.
- **@shopify/react-native-skia** — for the scratch-to-reveal mask (see Interactions). This
  is the cleanest native equivalent of the HTML canvas `destination-out` erase.
- **react-native-reanimated** — float/blink/pulse animations.
- **expo-font** + **@expo-google-fonts/anton**, **/outfit**, **/space-mono** — typefaces.
- **expo-linear-gradient** — all the gradient fills (RN `backgroundColor` can't do gradients).
- **expo-blur** (`BlurView`) — the tab bar's frosted-glass look (replaces CSS `backdrop-filter`).
- **expo-auth-session** / **expo-apple-authentication** — real Apple/Google sign-in later.

---

## Design Tokens

Put these in `theme/tokens.ts`.

### Colors
| Token | Hex | Use |
|---|---|---|
| `bgDeep` | `#04100b` | darkest background stop |
| `bgFelt` | `#061712` / `#0a2a1d` | felt-green gradient stops |
| `feltGlow` | `rgba(20,80,54,0.65)` | top radial glow |
| `gold` | `#FFC83D` | primary accent |
| `goldBright` | `#FFE39A` | bright gold text |
| `goldDeep` | `#C8922A` | gold gradient dark stop |
| `gold24` | `#FFD24A` | gold gradient light stop (medals/buttons) |
| `cyan` | `#34E4D6` | secondary neon (links, state picker) |
| `magenta` | `#FF4D9D` | tertiary neon (Research accent, 24/7 stat) |
| `red` | `#FF3B52` | brand red ("JACK", YouTube/subscribe) |
| `redSoft` | `#FF8AA0` / `#FF6B7A` | streak count, negative EV |
| `navy` | `#16265E` | brand navy (card gradients) |
| `green` | `#38E08A` | positive EV, success/earned |
| `purple` | `#9D7CFF` | "Behind the Scenes" category chip |
| `textPrimary` | `#F6F1E4` | warm white body text |
| `textOnLight` | `#0a1f17` | text on gold surfaces |
| `muted` | `#8FA89A` | secondary labels |
| `mutedDim` | `#6f8a7c` / `#5d7a6c` | tertiary / inactive tab |
| `bodyText` | `#A9C0B3` / `#C4D6CB` | paragraph copy |

### Gradients
- **App background** (vertical, top→bottom): radial `feltGlow → transparent` at top, over
  linear `#0a2a1d 0% → #061712 55% → #04100b 100%`.
- **Page backdrop (outside device, web only)** — N/A on native.
- **Gold fill** (145°): `#FFD24A → #C8922A`.
- **EV bar, positive** (90°): `#38E08A → #FFC83D`. **negative**: `#C8283A → #FF6B7A`.
- **XP bar** (90°): `#34E4D6 → #FFC83D`, glow `0 0 12px rgba(52,228,214,0.6)`.
- **Player/stat cards** (145°): `rgba(22,38,94,0.5) → rgba(10,42,29,0.55)`.
- **Medal #1**: `#FFD24A → #C8922A`; **#2**: `#E2E8F0 → #9AA6B2`; **#3**: `#E0964A → #B36A28`.

### Typography
Three families, loaded via `@expo-google-fonts`:
- **Anton** (`Anton_400Regular`) — display/marquee: screen titles (36px), big numbers,
  medal digits, hero ticket name, wordmark. Letter-spacing ~0.5px.
- **Outfit** (400/500/600/700/800) — all UI text, buttons, ticket names, body.
- **Space Mono** (400/700) — numeric/label text: stats, odds, EV labels, eyebrow labels,
  meta lines. Often `letter-spacing: 1–1.5px`, uppercase.

Scale (px = RN units): screen title 36 / hero name 25–26 / big stat 24–26 /
section title 16 / body 13–15 / meta & labels 9–11.

### Wordmark
"SCRATCH" in `textPrimary` (#fff) + "JACK" in `red` (#FF3B52), Anton, no ".com".

### Radius / Spacing / Shadow
- Radius: chips/pills `999px`; cards `14–18px`; large hero `18px`; inputs `12px`;
  medal/level tiles `9–13px`; logo container none (crest is transparent art).
- Screen padding: `14px 18px 28px`. Card padding: `12–17px`. Gap between list cards: `10px`.
- Neon glow pattern: `boxShadow: 0 0 Npx rgba(<accent>, 0.2–0.6)` → RN `shadowColor` +
  `shadowOpacity`/`shadowRadius` (iOS) and `elevation` (Android), or a Skia/blur halo.
- Tab bar top: `border-top: 1px rgba(255,200,61,0.18)`, shadow `0 -8px 26px rgba(0,0,0,0.55)`.

---

## Screens / Views

### 0. Login / Sign-up  (`login.tsx`)
- **Purpose**: Gate the app; user logs in or creates an account.
- **Layout**: Full-screen, vertically centered column, padding `44px 26px`, on the app
  background gradient. Scrollable.
- **Components (top→bottom)**:
  - Crest logo `118×118`, `objectFit: contain`, gold drop-shadow glow, gentle float
    animation (translateY ±3px, 4s loop).
  - Wordmark, Anton 32px.
  - Subtitle, Space Mono 13px, `muted`. Text switches with mode:
    sign-up → "Create your free account"; login → "Welcome back, high roller".
  - **Segmented toggle** ("Log In" | "Sign Up"): container `rgba(0,0,0,0.3)`, border
    `rgba(255,200,61,0.2)`, radius 13, padding 4. Active segment = gold gradient fill,
    text `#0a1f17`, weight 800; inactive = transparent, `muted`.
  - **Inputs** (radius 12, bg `rgba(255,255,255,0.05)`, border `rgba(255,255,255,0.14)`,
    padding `14×16`, text #fff, Outfit 15): "Display name" (sign-up only), "Email address",
    "Password" (secureTextEntry).
  - **Primary button** "Log In" / "Create Account": full width, gold gradient, text
    `#0a1f17` weight 800, radius 13, gold glow.
  - Divider with centered "OR" (Space Mono 10px).
  - **Apple** button (white apple glyph + "Continue with Apple") and **Google** button
    (white circle "G" in #4285F4 + "Continue with Google"): bg `rgba(255,255,255,0.06)`,
    border `rgba(255,255,255,0.14)`, radius 12, text #fff weight 700.
  - Footer: "21+ · PLAY RESPONSIBLY" pill (red outline) + "By continuing you agree to our
    Terms & Privacy Policy."
- **Behavior**: In the prototype any of the three buttons (primary / Apple / Google) sets
  `authed = true` and routes into the tabs. In production, wire to real auth providers; the
  segmented toggle only swaps copy + shows/hides the name field.

### 1. Rankings  (`rankings.tsx`) — default tab
- **Purpose**: The core value — EV-ranked scratch tickets for the selected state, plus the
  daily gamification hub.
- **Layout**: Scroll view. Order: title block → state picker → player/streak card →
  Top Bet scratch hero → full rankings list.
- **Components**:
  - **Title block**: "RANKINGS" Anton 36px, cyan text-glow; subtitle "EV-RANKED SCRATCHERS ·
    UPDATED TODAY" Space Mono 11px muted.
  - **State picker**: full-width button, bg `rgba(22,38,94,0.55)`, border `rgba(52,228,214,0.35)`,
    radius 14, padding `13×16`. Left: cyan pin icon + state name (Outfit 700 16). Right:
    "CHANGE ▾" cyan Space Mono. Tapping opens a dropdown (absolute, `#0a2418` bg, cyan border,
    max-height ~230 scroll) listing 12 states; selecting sets the state and closes.
  - **Player / streak card**: gradient navy→felt, gold border, radius 18, padding 16.
    - Row: level medal tile (46×46, gold gradient, Anton level number, glow) + title
      ("High Roller") + "LVL 7 · 1,240 / 2,000 XP" (Space Mono 10). Right: streak count
      (Anton 26 `redSoft`) + "DAY STREAK".
    - XP progress bar: track `rgba(0,0,0,0.35)` h7 radius99; fill = XP% (62%), cyan→gold
      gradient + cyan glow.
    - Divider (1px `rgba(255,255,255,0.08)`).
    - **Daily check-in**: "DAILY CHECK-IN" label + 7 dots (26×26). Past days = gold gradient
      with ✓; today = transparent w/ gold ring + glow showing day number; future = `rgba(255,255,255,0.06)`.
      Claim button: default gold gradient "CLAIM +250 ★" → on tap becomes green
      "CLAIMED ✓" (disabled) and adds 250 coins.
  - **Top Bet scratch hero** (see Interactions for the scratch mechanic):
    - Header row: "★ TODAY'S TOP BET" (Anton 14, gold, glow) + blinking "SCRATCH TO REVEAL"
      (cyan, Space Mono, opacity blink 1.8s).
    - Reward panel: radius 18, gold border `rgba(255,200,61,0.5)`, gold glow, min-height 164,
      bg gradient `#0c3322 → #06231a`. Content (revealed underneath the foil): eyebrow
      "{STATE} · #1 RANKED" (cyan mono 10), hero name "$30 · 500X LOTERIA" (Anton 25), and
      three stat columns — Net EV `+8.2%` (Anton 24 green), Top Prizes Left `4 of 6` (white),
      Win Odds `1 in 3.21` (gold).
    - On reveal: a floating green pill "✓ REVEALED +50 ★" appears top-right; +50 coins added.
  - **Full rankings list**: label "FULL RANKINGS · {STATE}". 8 ticket rows (`TicketRow`):
    - Rank medal tile (34×34, Anton 16; top-3 use medal gradients, rest `rgba(255,255,255,0.08)`).
    - Middle: ticket name (Outfit 700 15, ellipsis) + meta "${price} · {odds} · {left} left"
      (Space Mono 10 muted) + thin EV bar (h4, fill width = `(ev+8)/17`, pos/neg gradient).
    - Right: EV value (Anton 18; green if ≥0 else `#FF6B7A`) + "NET EV" caption.
    - **Rank #1 row** is emphasized: gold-tinted bg, gold border, gold glow.

### 2. Videos  (`videos.tsx`)
- **Purpose**: Surface the @ScratchJackOfficial YouTube uploads.
- **Layout**: Scroll. Title → channel bar → featured upload → 2-col grid of episodes.
- **Components**:
  - Title "VIDEOS" Anton 36, red glow; subtitle "FRESH FROM THE CHANNEL".
  - **Channel bar**: crest 42 + "@ScratchJackOfficial" (Outfit 700) + "YouTube · 142K
    subscribers" (mono 10) + red "SUBSCRIBE" pill (links to
    `https://www.youtube.com/@ScratchJackOfficial`). bg `rgba(255,59,82,0.1)`, red border.
  - **Featured** card: 16:9-ish thumbnail (h190, radius14), play button (58px white circle,
    red triangle), "NEW" red chip top-left, duration chip bottom-right, gradient scrim.
    Title (Outfit 700 16) + meta "142K views · 2 days ago".
  - **Grid** (2 columns, gap 13): each `VideoCard` = thumbnail (h96) with small play button +
    duration chip, 2-line title, "{views} views".
  - **Placeholders**: thumbnails are diagonal-striped gradient placeholders (cycle 6 tint
    pairs) — replace with real YouTube thumbnails. All cards currently deep-link to the channel.
- **Production note**: pull real uploads via the **YouTube Data API v3** (`search.list` /
  `playlistItems` for the uploads playlist) → thumbnail, title, duration, viewCount.

### 3. Guide  (`guide.tsx`)
- **Purpose**: Explain how the rankings work; FAQ.
- **Layout**: Scroll. Title "GUIDE" (subtitle "HOW SCRATCHJACK BEATS THE ODDS") → 4
  numbered step cards → "FREQUENTLY ASKED" accordion (3 items).
- **Components**:
  - **Step card**: number tile (40×40, per-step gradient: gold/cyan/magenta/green + glow) +
    title (Outfit 700 16) + description (13, `bodyText`). Copy is in the data table below.
  - **FAQ item**: bg `rgba(22,38,94,0.4)`, cyan border, radius 13. Header button (Outfit 600
    14) with cyan +/− on the right; expands to reveal answer (13, `bodyText`). One open at a time.

### 4. Research  (`research.tsx`)  *(was "Blog")*
- **Purpose**: Long-form strategy/methodology posts.
- **Layout**: Scroll. Title "RESEARCH" (magenta glow), subtitle "DATA, METHODOLOGY & DEEP
  DIVES" → vertical list of 5 post cards.
- **Components**: `PostCard` = left thumbnail (72×72 striped placeholder) + category chip
  (colored by category: Strategy=cyan, Odds=gold, Rankings=magenta, Behind the Scenes=purple;
  chip text `#0a1f17`) + title (Outfit 700 14) + "{date} · {read}" (mono 10). Cards link to
  `https://www.scratchjack.com/`.

### 5. Rewards  (`rewards.tsx`)  *(was "About")*
- **Purpose**: Brand/identity + the player's gamification status (badges) + links + legal.
- **Layout**: Scroll, centered hero. Crest 130 (float anim) → wordmark 30 → tagline
  "Play the odds, not the hype." → story paragraph card → 3-up stat row → "YOUR BADGES"
  grid → action buttons → 21+ legal footer.
- **Components**:
  - Story card: bg `rgba(255,255,255,0.04)`, radius 16, body copy (see data).
  - **Stats** (3 tiles): "12 STATES" (gold), "480+ TICKETS" (cyan), "24/7 UPDATES" (magenta) —
    Anton 24 number + mono caption, each in a navy→felt gradient tile with tinted border.
  - **Badges** grid (2 col): circular icon (36) + name + status. Earned = gold gradient icon,
    "EARNED", full opacity, gold-tint card; locked = dim (opacity 0.45), "LOCKED". Badges:
    First Scratch (★, earned), 7-Day Streak (7, earned), High Roller (♦, earned),
    Odds Master (%, locked).
  - **Buttons**: red "Subscribe on YouTube" (YT glyph) → channel URL; gold-outline
    "Visit ScratchJack.com" → site URL.
  - **Footer**: "21+ · PLAY RESPONSIBLY" pill + "Gambling problem? Call 1-800-GAMBLER.
    ScratchJack is an informational tool and does not sell tickets."

---

## Bottom Tab Bar (persistent, app only)
Five tabs, frosted dark glass (`expo-blur` over `rgba(4,16,11,0.94)`), top gold hairline,
upward shadow, bottom safe-area inset (the prototype reserves 22px for the home indicator).
Each tab = icon (22px, line/solid) + label (Outfit 600 10). **Active** = gold `#FFC83D`,
glow, slight translateY(-1px); **inactive** = `mutedDim` `#5d7a6c`.

| Order | Label | Icon (described) |
|---|---|---|
| 1 | Rankings | three ascending bars (podium) |
| 2 | Videos | rounded rect with play triangle |
| 3 | Guide | circle with "?" |
| 4 | Research | document with text lines |
| 5 | Rewards | circle with "i" |

(Recreate icons with `react-native-svg` or a line-icon set like Lucide; match the metaphors.)

---

## Interactions & Behavior

### Scratch-to-reveal (Top Bet)
The web prototype overlays an HTML `<canvas>` "foil" on the reward panel and erases it with
`globalCompositeOperation = 'destination-out'` on pointer move; when >~28% of pixels are
cleared it auto-completes (clears the foil, sets `scratched`, +50 coins, shows the REVEALED
pill). Brush radius is large (≈46px on a 660×328 backing) for an easy, low-effort scratch.

**Native implementation** (recommended, **react-native-skia**):
- Render the reward content, then a Skia `<Canvas>` foil on top filled with the gold
  gradient + "SCRATCH HERE / ★ reveal the top bet ★" text + sparkle dots.
- Track touches via `useTouchHandler`; accumulate erase circles in a `Path`/mask and paint
  them with `BlendMode.Clear` so the reward shows through.
- Estimate cleared area from touch coverage (or sample); at ≥~28–30% run the completion:
  fade/clear the foil, set `scratched=true`, award +50 coins, animate in the pill.
- Keep the brush generous (~40–46 dp) so it feels effortless.
- Once revealed, stays revealed (don't reset on state change).

### Daily check-in / Claim
Tap Claim → `claimed=true`, today's dot flips to ✓, +250 coins, button → green "CLAIMED ✓"
(disabled). Idempotent.

### State picker
Tap → toggle dropdown; pick a state → set `stateName`, close dropdown. In production the
selected state drives which ranking dataset is fetched.

### FAQ accordion
One item open at a time; tapping the open item closes it. +/− indicator.

### Animations (use Reanimated)
- **Float** (logo/crest, REVEALED pill): translateY 0 → -3px → 0, ~2.5–4s ease-in-out loop.
- **Blink** ("SCRATCH TO REVEAL"): opacity 1 ↔ 0.35, 1.8s loop.
- Tab press: active tab nudges up 1px + gains gold glow.

### Navigation
Auth gate: unauthenticated → `login`; authenticated → `(tabs)` with Rankings focused.
External links (Subscribe, Visit site, video/post cards) open in the browser
(`expo-linking` / `expo-web-browser`).

---

## State Management

Local/UI state (e.g. a `usePlayer` store via Zustand or React context):
- `authed: boolean`, `authMode: 'login' | 'signup'`
- `stateName: string` (selected state; default "Texas")
- `pickerOpen: boolean`
- `scratched: boolean`, `claimed: boolean`
- `coins: number` (default 2450; +50 on reveal, +250 on claim)
- `streak: number` (12), `level: number` (7), `xpPct` (62), `faqOpen: number | null`

Server-backed (build these APIs):
- **Auth** — Apple/Google/email → user profile; persist coins/streak/level/badges/claims
  server-side so they survive reinstall and sync across devices.
- **Rankings feed** — `GET /rankings?state=TX` → ranked tickets (see shape below). Computed
  by a backend job that scrapes/licenses each state lottery's prizes-remaining data daily and
  calculates Net EV = f(top prizes remaining ÷ tickets remaining, ticket price, prize table).
- **Videos** — proxy the YouTube Data API (cache server-side; don't ship the API key in-app).
- **Daily check-in** — server validates the streak and grants the reward (prevent client clock abuse).

---

## Sample Data (exact values from the prototype)

### States (picker)
Texas, Florida, New York, California, Ohio, Georgia, Pennsylvania, New Jersey, Illinois,
Arizona, Michigan, Virginia.

### Tickets (Rankings list; rank, name, price, ev%, odds, top-prizes-left)
1. 500X Loteria — $30 — +8.2% — 1 in 3.21 — 4 of 6
2. Ultimate Millions — $50 — +6.7% — 1 in 3.04 — 2 of 4
3. Gold Rush Tripler — $25 — +5.1% — 1 in 3.55 — 7 of 10
4. Lucky 7s Bonus — $20 — +2.9% — 1 in 3.78 — 12 of 20
5. Neon Cash Blowout — $10 — +0.4% — 1 in 4.10 — 30 of 50
6. Diamond Dazzler — $5 — −1.8% — 1 in 4.66 — 88 of 120
7. Triple Cherry — $3 — −3.2% — 1 in 4.90 — 210 of 300
8. Quick Spin — $2 — −5.6% — 1 in 5.20 — 540 of 800

EV bar fill = `clamp(5, (ev + 8) / 17 * 100, 100)` %. Color green if ev ≥ 0 else red.

### Featured video
"I Bought $1,000 in Texas Scratch-Offs — Here's What Happened" — 18:24 — 142K views · 2 days ago.

### Videos grid (title, duration, views)
- $50 Tickets ONLY — Did We Actually Profit? — 12:05 — 88K
- Top 5 Scratchers to AVOID Right Now — 9:41 — 210K
- Claiming a $10,000 Winner LIVE — 22:13 — 305K
- Florida vs Texas — Who Has Better Odds? — 14:30 — 67K
- How I Read the Prizes-Remaining Data — 7:58 — 54K
- $5 Challenge: Turning $100 Into a Win — 16:02 — 120K

### Guide steps
1. **Pick your state** — We pull live prize data straight from your state lottery the moment it updates.
2. **We crunch the odds** — Remaining top prizes divided by tickets left tells us what each game is really worth today.
3. **Every ticket gets ranked** — A single Net EV score scores each scratcher on its true expected payout — apples to apples.
4. **Play smarter** — Chase the green, skip the red. Spend on the games that still have winners in them.

### FAQ
- **What does "Net EV" mean?** — Net EV is the expected return per dollar after accounting for
  the prizes still unclaimed. Positive means the game currently pays back more than average;
  negative means the best prizes are mostly gone.
- **How often is the data updated?** — Prize-remaining counts refresh automatically every day,
  and rankings re-sort the moment a top prize is claimed.
- **Does ScratchJack sell tickets?** — No. We are an informational odds tracker only. Buy
  tickets through your official state lottery retailers.

### Research posts (title, category, date, read)
- The Real Math Behind 'Prizes Remaining' — Strategy — Jun 12 — 6 min read
- Why $30 Tickets Usually Beat $5 Tickets — Odds — Jun 5 — 8 min read
- State-by-State: Best Scratch-Off Odds 2026 — Rankings — May 28 — 11 min read
- Bankroll Management for Scratch Players — Strategy — May 20 — 5 min read
- How We Calculate the ScratchJack EV Score — Behind the Scenes — May 10 — 9 min read

### Rewards copy
- Tagline: "Play the odds, not the hype."
- Story: "ScratchJack tracks every state lottery's scratch-off prizes in real time, then ranks
  each ticket by its true expected value — so you always know which games still have the big
  winners waiting, and which are dead. No hype. Just the math."
- Stats: 12 STATES · 480+ TICKETS · 24/7 UPDATES
- Badges: First Scratch (earned), 7-Day Streak (earned), High Roller (earned), Odds Master (locked)
- Legal: "21+ · PLAY RESPONSIBLY" · "Gambling problem? Call 1-800-GAMBLER. ScratchJack is an
  informational tool and does not sell tickets."

### External URLs
- YouTube: `https://www.youtube.com/@ScratchJackOfficial`
- Website: `https://www.scratchjack.com/`

---

## Assets
- `design/assets/scratchjack-crest.webp` — **primary logo**, transparent king-on-shield crest,
  2080×2048, no text. Use everywhere (login, header, channel bar, Rewards hero). It glows on
  dark; no background plate needed. Generate a 1024² PNG app icon + adaptive icon from it.
- `design/assets/scratchjack-logo.jpg` — older square logo *with* ".com" text on white. Kept
  for reference only; prefer the crest.
- Video/post thumbnails in the prototype are striped gradient placeholders — replace with real
  YouTube thumbnails / post hero images.
- Fonts: Anton, Outfit, Space Mono (Google Fonts; bundle via `@expo-google-fonts`).

## Screenshots
Rendered reference shots of every screen (iPhone frame is preview-only — ignore the bezel):
- `screenshots/01-login.png` — Login / Sign-up
- `screenshots/02-rankings.png` — Rankings (default tab)
- `screenshots/03-videos.png` — Videos
- `screenshots/04-guide.png` — Guide
- `screenshots/05-research.png` — Research
- `screenshots/06-rewards.png` — Rewards

## Files
- `design/ScratchJack App.dc.html` — the full interactive prototype (all screens + logic).
  Source of truth. Open it in a browser to see every interaction and exact styling.
- `design/ios-frame.jsx` — preview-only device bezel; **do not port**.
- `design/assets/` — logo assets described above.

## Compliance reminder
App Store / Play Store enforce strict rules for gambling-adjacent apps. ScratchJack is an
**informational tool that does not sell tickets** — keep that framing, ship age-gating (21+),
responsible-gambling links (1-800-GAMBLER), and a clear "not affiliated with any state lottery"
disclaimer. Review the latest store policies before submission.
