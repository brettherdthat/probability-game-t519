# AI Development Guide - Dice Probability Game

## Project Overview

**Purpose:** Educational game teaching probability concepts to students through hands-on dice rolling experience.

**Tech Stack:** Vanilla JavaScript, HTML5, CSS3, MediaPipe Hands (for hand tracking)

**Live Demo:** https://github.com/brettherdthat/probability-game-t519

---

## Core Concept

Students learn probability by:
1. **Round 1:** Roll dice with chips evenly distributed (3 per column 2-12)
2. Experience which numbers appear more frequently
3. **Round 2:** Strategically redistribute chips based on learning
4. Compare results to understand normal distribution

**Key Insight:** Sum of two dice follows a normal distribution (7 is most common, 2 and 12 are rarest)

---

## File Structure

```
probability-game-t519/
├── index.html          # Complete single-file application
├── AI_DEVELOPMENT_GUIDE.md  # This file
└── [image files]       # Legacy assets from previous game version
```

**Note:** This is a single-page application. All HTML, CSS, and JavaScript are in `index.html`.

---

## Architecture Overview

### Game State Management

```javascript
const gameState = {
    round: 1,              // Current round (1 or 2)
    rollCount: 0,          // Total rolls in current round
    round1Rolls: 0,        // Rolls in round 1 (for comparison)
    round2Rolls: 0,        // Rolls in round 2 (for comparison)
    chips: {},             // Object mapping column (2-12) to chip count
    gameStarted: false,    // Whether game is active
    die1Value: null,       // Current die 1 value
    die2Value: null,       // Current die 2 value
    die1Rolled: false,     // Whether die 1 has been rolled
    die2Rolled: false,     // Whether die 2 has been rolled
    currentSum: 0,         // Current sum of both dice
    handTracking: {
        left: { prevY: 0, throwDetected: false, lastThrowTime: 0 },
        right: { prevY: 0, throwDetected: false, lastThrowTime: 0 }
    }
};
```

### Hand Tracking System

**Library:** MediaPipe Hands
- Tracks 2 hands independently
- Left hand controls left die
- Right hand controls right die
- Throwing gesture: rapid downward movement (velocity > 0.08)

**Key Functions:**
- `initCamera()` - Sets up webcam and MediaPipe
- `detectThrow(hand, landmarks)` - Detects throwing gesture
- `rollDie(dieNumber)` - Rolls individual die (1 or 2)

### Game Flow

```
Start Screen
    ↓
Round 1 (even distribution)
    ↓
Roll dice until all chips cleared
    ↓
Transition Screen (shows roll count)
    ↓
Redistribution UI (player places 33 chips)
    ↓
Round 2 (custom distribution)
    ↓
Roll dice until all chips cleared
    ↓
Victory Screen (compares Round 1 vs Round 2)
```

---

## Key Components

### 1. Game Board

**Location:** `#gameBoard` div
**Rendering:** `renderBoard()` function

Each column (2-12) displays:
- Column number at top
- Stack of chips (visual representation)
- Chips animate when created

### 2. Dice System

**Two-Hand Independent Rolling:**
- Left hand throws → rolls die 1
- Right hand throws → rolls die 2
- When both rolled → auto-calculates sum and removes chip
- 50ms delay prevents simultaneous processing

**Functions:**
- `rollDie(dieNumber)` - Rolls single die with animation
- `processRoll()` - Called when both dice ready
- `getDiceFace(value)` - Returns Unicode dice emoji

### 3. Hand Tracking

**Gesture Detection:**
```javascript
function detectThrow(hand, landmarks) {
    const wrist = landmarks[0];
    const currentY = wrist.y;
    const prevY = gameState.handTracking[hand].prevY;
    const velocity = currentY - prevY;

    // Throwing detected: rapid downward movement
    const isThrow = velocity > 0.08;

    gameState.handTracking[hand].prevY = currentY;
    return isThrow;
}
```

**Cooldown:** 800ms between throws per hand

### 4. Redistribution UI

**Purpose:** Let students strategically place chips for Round 2

**Constraints:**
- Must place exactly 33 chips
- Chips distributed across columns 2-12
- +/- buttons for each column
- Can't start Round 2 until all chips placed

**Functions:**
- `showRedistributionUI()` - Creates UI
- `addChip(column)` - Adds chip to column
- `removeRedistChip(column)` - Removes chip from column
- `updateRemainingChips()` - Updates counter and validation

---

## Common Modifications

### Change Number of Chips

**File:** `index.html`
**Function:** `initializeRound1Chips()`

```javascript
function initializeRound1Chips() {
    gameState.chips = {};
    for (let i = 2; i <= 12; i++) {
        gameState.chips[i] = 3; // ← Change this number
    }
}
```

**Also update:** Total chips in `showRedistributionUI()` (currently 33)

### Adjust Gesture Sensitivity

**File:** `index.html`
**Function:** `detectThrow()`

```javascript
const isThrow = velocity > 0.08; // ← Lower = more sensitive
```

### Change Cooldown Between Rolls

**File:** `index.html`
**Function:** `initCamera()` in `hands.onResults()`

```javascript
if (isThrow && now - gameState.handTracking[hand].lastThrowTime > 800) {
    // ← Change 800 (milliseconds)
}
```

### Modify Dice Animation Speed

**File:** `index.html`
**Function:** `rollDie()`

```javascript
setTimeout(() => {
    // ... dice result display
}, 400); // ← Change animation duration
```

### Add New UI Elements

**Pattern:**
1. Add HTML element in appropriate section
2. Add CSS styling in `<style>` tag
3. Add JavaScript event listener at bottom of `<script>` tag
4. Create handler function

**Example:** See `#restartRoundButton` implementation

---

## Important Functions Reference

### Core Game Loop

- `startGame()` - Entry point, not currently used (starts from startRound1)
- `startRound1()` - Initialize Round 1
- `startRound2()` - Initialize Round 2
- `startGameplay()` - Common initialization for both rounds
- `endRound()` - Handle round completion
- `isRoundComplete()` - Check if all chips cleared

### Dice Management

- `rollDie(dieNumber)` - Roll individual die
- `processRoll()` - Process completed roll (both dice)
- `updateDiceDisplay()` - Update visual state of dice
- `resetDiceState()` - Reset for next roll

### Board & Chips

- `renderBoard()` - Render all columns and chips
- `removeChip(column)` - Remove chip from column
- `updateRollCounter()` - Update roll count display

### Hand Tracking

- `initCamera()` - Initialize MediaPipe and webcam
- `detectThrow(hand, landmarks)` - Detect throwing gesture
- `onHandsDetected(results)` - MediaPipe callback (inline in initCamera)

### UI Screens

- `showVictoryScreen()` - Display final results
- `showRedistributionUI()` - Show chip placement interface

---

## State Transitions

### Dice State Machine

```
Not Rolled (?, ?)
    ↓ (throw left hand)
Die 1 Rolled (⚀-⚅, ?)
    ↓ (throw right hand)
Both Rolled (⚀-⚅, ⚀-⚅)
    ↓ (auto after 300ms)
Process Roll → Remove Chip
    ↓
Reset (?, ?)
```

### Round State Machine

```
Round 1 Start (even distribution)
    ↓
Playing (rolling dice)
    ↓
All Chips Cleared
    ↓
Transition Screen
    ↓
Redistribution UI
    ↓
Round 2 Start (custom distribution)
    ↓
Playing (rolling dice)
    ↓
All Chips Cleared
    ↓
Victory Screen
```

---

## UI Component Layout

```
┌─────────────────────────────────────────────────────┐
│ [Rolls: X]  [Instructions]      [Round 1] [Webcam] │
│                                                      │
│                  [Sum: ?]                           │
│                [Die] [Die]                          │
│                                                      │
│  [2] [3] [4] [5] [6] [7] [8] [9] [10] [11] [12]   │
│  [●] [●] [●] [●] [●] [●] [●] [●] [●]  [●]  [●]    │
│  [●] [●] [●] [●] [●] [●] [●] [●] [●]  [●]  [●]    │
│  [●] [●] [●] [●] [●] [●] [●] [●] [●]  [●]  [●]    │
└─────────────────────────────────────────────────────┘
```

**Z-Index Layers:**
1. Background gradient
2. Game board
3. Dice area
4. UI elements (counters, buttons)
5. Instructions box
6. Hand tracking canvas overlay
7. Modal screens (start, transition, victory, redistribution)

---

## MediaPipe Configuration

```javascript
hands.setOptions({
    maxNumHands: 2,              // Track both hands
    modelComplexity: 1,          // Medium complexity
    minDetectionConfidence: 0.6, // 60% confidence threshold
    minTrackingConfidence: 0.6   // 60% tracking threshold
});
```

**Landmarks Used:**
- `landmarks[0]` - Wrist position (for throw detection)
- Full hand tracking available but only wrist used

**Camera Settings:**
- Resolution: 640x480
- Mirrored display (scaleX: -1)
- Positioned top-right, 200x150px

---

## Known Limitations & Considerations

### Hand Tracking
- **Lighting dependent** - Poor lighting affects detection
- **Gesture sensitivity** - May need adjustment per user
- **Cooldown needed** - Prevents double-triggering
- **Single hand mode broken** - Stick with two-hand version

### Game Logic
- Round 2 restart doesn't preserve custom distribution (resets to 3 each)
- No persistence - refresh loses progress
- No validation that dice sums are statistically correct over time

### Browser Compatibility
- Requires camera access
- MediaPipe may not work in all browsers
- Tested primarily in Chrome

---

## Debugging Tips

### Hand Tracking Issues

**Problem:** Hand not detected
- Check `handCtx` visualization (yellow/green dots)
- Verify camera permissions granted
- Adjust `minDetectionConfidence` lower
- Ensure good lighting

**Problem:** Gesture not triggering
- Lower `velocity` threshold in `detectThrow()`
- Check cooldown isn't too long
- Verify `gameState.gameStarted === true`

### Dice Not Rolling

**Problem:** Dice stay at "?"
- Check `die1Rolled` and `die2Rolled` states
- Verify `rollDie()` is being called
- Check console for errors
- Ensure `processRoll()` triggers after both rolled

### UI Issues

**Problem:** Elements not visible
- Check z-index values
- Verify `.hidden` class not applied
- Check positioning (absolute vs relative)

---

## Git Workflow

**Main Branch:** `main`

**Recent Commits:**
1. Initial probability game transformation
2. Two-hand dice rolling implementation
3. Compact instructions box
4. Restart round button

**Commit Message Format:**
```
Brief description (imperative mood)

Changes:
- Bullet point 1
- Bullet point 2

🤖 Generated with Claude Code
Co-Authored-By: Claude <noreply@anthropic.com>
```

---

## Future Enhancement Ideas

### Suggested Features
1. **Statistics tracking** - Show probability chart after rounds
2. **Difficulty levels** - Different chip counts
3. **Multiplayer** - Compare strategies between students
4. **Sound effects** - Dice rolling sounds, chip removal
5. **Tutorial mode** - Guided first playthrough
6. **Save/load** - Preserve custom distributions
7. **Click mode** - Alternative to hand tracking
8. **Probability hints** - Show expected frequencies

### Code Improvements
1. **Separate files** - Split HTML/CSS/JS
2. **Module system** - Break into components
3. **State management** - Use reducer pattern
4. **Testing** - Unit tests for game logic
5. **Accessibility** - Screen reader support, keyboard controls
6. **Responsive design** - Mobile optimization

---

## Quick Reference Commands

### Testing Locally
```bash
open index.html  # macOS
start index.html # Windows
```

### Git Operations
```bash
git add index.html
git commit -m "Description"
git push
```

### Chrome DevTools
- `Cmd+Option+I` (macOS) / `F12` (Windows) - Open DevTools
- Check Console for errors
- Use Sources tab to debug JavaScript
- Network tab to verify MediaPipe loads

---

## Contact & Collaboration

**Repository:** https://github.com/brettherdthat/probability-game-t519

**Educational Context:** Designed for classroom use teaching probability and statistics

**AI Collaboration:** This document designed to help AI assistants understand and modify the codebase effectively

---

## Appendix: Complete Function List

### Game Flow Functions
- `initializeRound1Chips()`
- `startRound1()`
- `startRound2()`
- `startGameplay()`
- `endRound()`
- `isRoundComplete()`
- `showVictoryScreen()`

### Dice Functions
- `rollDie(dieNumber)`
- `getDiceFace(value)`
- `updateDiceDisplay()`
- `processRoll()`
- `resetDiceState()`

### Board Functions
- `renderBoard()`
- `removeChip(column)`
- `updateRollCounter()`

### Hand Tracking Functions
- `initCamera()`
- `detectThrow(hand, landmarks)`

### Redistribution Functions
- `showRedistributionUI()`
- `addChip(column)`
- `removeRedistChip(column)`
- `updateRemainingChips()`

### Event Handlers
- Start button click
- Redistribute button click
- Start Round 2 button click
- Play Again button click
- Restart Round button click

---

**Document Version:** 1.0
**Last Updated:** 2024
**Maintained By:** AI-assisted development team
