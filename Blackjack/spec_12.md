# spec.md — Deep Sea Blackjack

## Visual Design
- **Theme:** Deep-ocean bioluminescent aesthetic — dark abyss fading to surface blue
- **Background:** Radial gradient (#020c16 → #0d6e9e) with animated caustic light ripples
- **Animated scene:** Rising bubbles, swaying kelp forest along bottom, shark gliding across, jellyfish bobbing at edges, fish parade (🐟🐠🐡) swimming across on wins
- **Currency:** Pearls (🦪 oyster emoji), rendered as iridescent chip tokens with 6 denominations
- **Cards:** Sea-glass blue-green faces with diagonal wave texture; ocean creature suits — 🐬 Dolphin, 🪸 Coral, 🐠 Clownfish, 🦑 Squid; deep-navy bioluminescent backs with animated glowing 🐚 shell
- **Win particles:** Rising iridescent pearl orbs with varied colors
- **Typography:** Cormorant Garamond (display), Nunito (UI), Cinzel (labels)
- **Responsive:** 360px mobile → 1920px desktop

## Layout
- Full-viewport ocean scene with layered z-index elements
- **HUD (fixed top):** Pearls balance | Wager | Title | Current Count | Shoe remaining
  - Mobile: Title on top row, 4 stats below in flex row
- **Main area:** Dealer zone (top) → Message strip (center) → Player zone (bottom)
- **Bet area:** Pearl chip tray (1/5/10/25/100/500) above action buttons
- **Side panel (collapsible):** Dive Stats, Strategy Advice, Dive Log

## Game States

| State | Chips | Deal | Hit/Stand | Double | Split | Clear |
|-------|-------|------|-----------|--------|-------|-------|
| BETTING | ✅ | ✅ (if bet>0) | ❌ | ❌ | ❌ | ✅ |
| PLAYING | ❌ | ❌ | ✅ | ✅ (if 2 cards + funds) | ✅ (if pair + funds) | ❌ |
| DONE | ❌ | ✅ "New Dive" | ❌ | ❌ | ❌ | ❌ |

## Blackjack Rules

| Rule | Implementation |
|------|----------------|
| Decks | 6-deck shoe (312 cards), reshuffled when < 52 remain |
| Dealer | Stands on soft 17+ |
| Blackjack | Pays 3:2 — `Math.floor(bet × 1.5)` |
| Push | Simultaneous blackjack → bet returned |
| Insurance | Dealer Ace → offer half-bet; pays 2:1 if dealer has BJ |
| Double Down | First 2 cards only; one additional card; per-hand wager tracked |
| Split | Equal ranks → two independent hands at half bet each |
| Split Aces | Exactly 1 card each, then auto-stand |
| Card counting | Hi-Lo: 2-6=+1, 7-9=0, 10-A=-1 (running count displayed) |

## Required Testing Scenarios

### Test 1 — Player Blackjack, Dealer No Blackjack
```
bet = 100
pay = floor(100 × 1.5) = 150
balance += bet + pay = +250
Expected message: "BLACKJACK! 🐬 + 🦪 150"
Expected result: WIN (bj)
```

### Test 2 — Both Player and Dealer Blackjack (Push)
```
bet = 100
balance += 100 (returned)
net = 0
Expected message: "PUSH — Both Blackjack! 🤝"
Expected result: PUSH
```

### Test 3 — Insurance Win (dealer has BJ, player took insurance)
```
bet = 100, insBet = 50
balance -= 50 (insurance cost)
dealer has BJ → balance += 50 × 3 = 150 (insurance pays 2:1)
main bet lost → net = -100 + 100 = 0 (break even)
Expected message: "Dealer Blackjack! 🦈 You lose. (Ins: +🦪 100)"
```

### Test 4 — Double Down Win
```
bet = 50 → doubleCost = 50 → bet becomes 100
win → balance += 100 × 2 = 200 → net = +100
```

### Test 5 — Split, Both Hands Win
```
bet = 100 → halfBet = 50 → balance -= 50
splitWagers = [50, 50]
both win → balance += 50×2 + 50×2 = 200 → net = +100
```

### Test 6 — Dealer Soft 17 (dealer must hit)
```
Dealer has A + 6 = soft 17 → dealer HITS
Dealer has A + 7 = soft 18 → dealer STANDS
```

## Reflection

**Q: How certain are you that blackjack rules work for all card combinations that result in 21?**

The `total()` function handles all cases correctly:
- Any Ace + any 10-value card (10, J, Q, K) = 21 → detected by `isBJ(hand)` which checks `hand.length===2 && total(hand)===21`
- Ace is counted as 11 first; reduced to 1 whenever total would exceed 21
- All 4 suits × 4 ranks (10/J/Q/K) × 4 ace suits = 64 unique blackjack combinations
- Soft hands handled throughout (isSoft() used for dealer decisions and strategy hints)

**Q: Have you tested all possible blackjack combinations?**

The logic is proven rather than enumerated — since `isBJ` depends only on `hand.length===2` and the rank-based `total()` function, and `cVal()` correctly maps J/Q/K/10 → 10, all 64 combinations return the same result.

**Q: How do you know the deck is complete and shuffled?**

`buildShoe()` constructs exactly 6 × 4 × 13 = **312 cards** via nested loops. `fyShuf()` implements the Fisher-Yates shuffle which guarantees each permutation is equally probable. The live shoe count in the HUD confirms the count. The shoe is reshuffled when fewer than 52 cards remain.

## Features Implemented

| Category | Feature |
|----------|---------|
| **Core Rules** | 6-deck shoe, dealer stands soft 17, BJ 3:2, push, insurance, double down, split, split aces |
| **UI** | Card deal + flip animations, win particles, fish parade, iridescent pearl chips |
| **Audio** | Card flip, pearl drop (chip), win fanfare, big win, loss (low whale), push, bust |
| **Enhancements** | Hi-Lo card counting, basic strategy hints, betting history, statistics |
| **Accessibility** | Keyboard shortcuts: H(hit), S(stand), D(deal/new), X(double), C(clear) |
| **Responsive** | Mobile-optimized HUD, hidden shortcuts, compact cards + buttons |
| **Polish** | Auto-refill at zero, animated shark/jellyfish/kelp scene, 3-state button logic |
