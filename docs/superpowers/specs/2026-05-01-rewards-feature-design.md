# Rewards Feature Design

**Date:** 2026-05-01  
**Feature:** Add a rewards system to the Pub Golf scoring app  
**Status:** Design approved, ready for implementation

---

## Overview

Add a rewards feature that mirrors the existing penalties system but with key differences:
- **Penalties:** Individual player wrongdoings (vomiting, spilling, etc.) that are divided among team members
- **Rewards:** Team achievements (completing a mission) that deduct directly from the team's total score

The "⚡ Penalties" tab will be renamed to "⚡ Rewards/Penalties" to accommodate both features.

---

## Data Structure

### State Changes

Add two new arrays to `state`:

```javascript
{
  rewards: [
    { name: string, value: number }
  ],
  rewardLog: [
    { teamName: string, rewardName: string, strokes: number }
  ]
}
```

**Rewards:** Array of reward definitions that the organizer creates during game setup.

**RewardLog:** Array of rewards awarded during gameplay. Each entry tracks:
- `teamName`: The team that earned the reward
- `rewardName`: Name of the reward (e.g., "Completing a hard secret mission")
- `strokes`: Value of the reward (strokes deducted from team score)

### Initialization

When a new game is created, initialize:
```javascript
rewards: [],
rewardLog: []
```

When a game is loaded from Firebase, apply the same snapshot logic used for penalties and penaltyLog.

---

## Lobby Setup - Reward Definition

### Location
Add a new **"Rewards"** section in the organizer's game setup screen, positioned **below** the penalties section.

### Components

**Input Fields:**
- Reward name input (text field, e.g., "Completing a hard secret mission")
- Reward value input (number field, positive integer)

**Controls:**
- "+ Add Reward" button to add new reward
- Remove button (✕) for each reward in the list

**Styling:**
- Match the existing penalties section styling
- Use same card layout, colors, and typography
- Display added rewards in a list with name and value visible

### Functions

- `renderOrgRewardSetup()` - Render the reward definition UI
- `addRewardDefLobby()` - Add a new reward to state and re-render
- `removeRewardDefLobby(idx)` - Remove a reward by index and re-render

### Firebase Sync

When a reward is added/removed, update Firebase with `patchState({ rewards: state.rewards })`.

---

## Game Flow - Award Rewards

### Location
In the game screen's "⚡ Rewards/Penalties" tab, add a **Rewards** section below the penalties section.

### Components

**Reward Award UI:**
- Team dropdown (organizer-only) - lists all teams in the game
- Reward type dropdown - lists all defined rewards with their values
- "Award Reward" button

**Reward Log Display:**
- List of awarded rewards showing:
  - Team name
  - Reward name
  - Reward value (strokes deducted)
  - Remove button (✕) - organizer-only

### Functions

- `logReward()` - Award a reward to a team
  1. Get selected team name from dropdown
  2. Get selected reward index from dropdown
  3. Retrieve reward from state.rewards
  4. Push entry to state.rewardLog: `{ teamName, rewardName, strokes }`
  5. Call `patchState({ rewardLog: state.rewardLog })`
  6. Re-render leaderboard and reward log

- `removeRewardLog(idx)` - Remove a logged reward
  1. Remove by index from state.rewardLog
  2. Call `patchState({ rewardLog: state.rewardLog })`
  3. Re-render leaderboard and reward log

- `renderRewardLog()` - Render the reward log list

---

## Score Calculation

### Formula

```
Team Total Score = Base Score - Total Penalties + Total Rewards
```

Where:
- **Base Score:** Sum of all hole scores for the team
- **Total Penalties:** Sum of all penalty strokes divided by team member count (existing behavior)
- **Total Rewards:** Sum of all reward strokes (new)

### Implementation

- `teamTotalRewards(teamName)` - New function to sum all rewards for a team
  - Iterate through `state.rewardLog`
  - Sum `strokes` where `teamName` matches
  - Return total

Update score calculation functions to include rewards:
- `teamTotalScore()` - already exists
- `teamTotalPar()` - already exists
- Use new `teamTotalRewards()` in leaderboard and final results calculations

---

## Display Changes

### Tab Rename

Change "⚡ Penalties" to "⚡ Rewards/Penalties" in the game screen tab bar.

### Leaderboard Display

In both the game leaderboard and final results leaderboard, display rewards and penalties separately:

**Format:**
- Penalties: `(+2 pen)` shown in red (#e74c3c)
- Rewards: `(-1 reward)` shown in green (#5ec988) — same color as "under par" to convey it as a positive
- Both shown on the same line as team name, separated

**Example:**
```
Team A (+2 pen, -1 reward)
```

### Leaderboard Calculation

When calculating `diff` (vs par) for leaderboard ranking:
```javascript
diff = r1(totalScore - totalPar)
```

The `totalScore` already includes penalty/reward adjustments, so ranking automatically reflects rewards.

---

## UI Changes Summary

1. **Lobby Setup:** Add "Rewards" section below "Penalties"
2. **Game Tab:** Rename tab to "Rewards/Penalties"
3. **Rewards/Penalties Tab:** Add organizer-only reward award UI
4. **Leaderboards:** Display rewards separately from penalties

---

## Error Handling

- Validate reward name is not empty
- Validate reward value is a positive number
- Prevent adding rewards if none are defined (show message)
- Prevent awarding reward if no teams exist

---

## Testing Considerations

- Verify rewards deduct correctly from team totals
- Verify rewards don't affect individual member scores
- Verify rewards display separately from penalties in leaderboard
- Verify organizer-only access to award rewards
- Verify Firebase syncs correctly
- Verify removal of rewards updates leaderboard immediately
- Test with multiple rewards awarded to same team
- Test with zero defined rewards (UI should handle gracefully)

---

## Implementation Notes

- Rewards structure parallels penalties but is simpler (no memberCount tracking)
- All Firebase reads/writes should use existing `patchState()` pattern
- Re-use `r1()` function for rounding reward display to 2 decimal places
- Styling should match penalties section for visual consistency
- Organizer-only UI elements should use same access control as penalties (check `local.isOrganiser`)
