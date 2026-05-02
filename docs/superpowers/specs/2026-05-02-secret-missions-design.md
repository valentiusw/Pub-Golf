# Secret Missions Feature Design

**Date:** 2026-05-02  
**Feature:** Add secret missions system to Pub Golf  
**Status:** Design approved, ready for implementation

---

## Overview

Add a Secret Missions feature where teams receive 2 easy and 1 hard mission during game setup. Missions are randomly assigned from a curated pool, with no two teams receiving the same mission. Teams track progress via local checkboxes (not synced to Firebase). The feature is toggleable by the organizer during game setup.

---

## Mission Pool

### Easy Missions (13 total)
1. Fist bump 5 players not from your team
2. Get a player from another team do a toast with your team
3. Take a selfie on the phone of a player from another team without them knowing
4. Get a player on another team to say the word 'wholeheartedly'
5. Get someone from another team to show their golf swing
6. Start an intra-team argument (between 2 players on the same team)
7. Get a player from another team to reveal how many steps they have done so far
8. Jump scare a player from another team (they must verbally scream)
9. Get a player from another team to download an app
10. Get a player from another team to call someone on speaker phone
11. Get a player on another team to spell the word 'Guiness'
12. Get a player on another team to teach you a word in a different language
13. Start a conga line with at least two players from another team

### Hard Missions (6 total)
1. Convince someone from another team to get the wrong drink
2. Make a player on another team say the word bro 4 times in one sentence
3. Get a player on another team to quote a book
4. Get a player on another team to do a pushup
5. Get a player on another team to reveal their secret challenges (revealer gets +2 stroke penalty)
6. Tie a player from another team's shoelaces together

---

## Data Structure

### State Changes

Add to `state`:
```javascript
{
  secretMissionsEnabled: boolean,
  missions: {
    [teamName]: [
      { text: string, difficulty: 'easy' | 'hard' },
      { text: string, difficulty: 'easy' | 'hard' },
      { text: string, difficulty: 'easy' | 'hard' }
    ]
  }
}
```

- `secretMissionsEnabled`: Toggle set by organizer during setup
- `missions`: Object mapping team names to their assigned 2 easy + 1 hard missions

### Local Storage

Checkbox state stored per team:
- Key: `{teamName}_missions_checked`
- Value: Array of mission texts that team has completed
- Persists across page reloads, survives team member leaving and rejoining

---

## Organizer Setup

### Location
Add below the Reward Setup section in the organizer's game setup screen.

### UI
```
🎯 Secret Missions
☐ Enable secret missions for this game

Description: Teams will receive 2 easy and 1 hard mission. 
Missions are assigned when teams join and cannot be shared.
```

### Functionality
- Toggle checkbox updates `state.secretMissionsEnabled`
- Changes sync to Firebase immediately via `patchState()`
- Can be toggled on/off before game starts
- Once game starts, toggling is disabled (no UI change needed—just document this)

---

## Mission Assignment

### Trigger
When a team joins the game (in `joinGame()` function).

### Logic
1. Check if `secretMissionsEnabled === true`
   - If false, skip assignment
2. Get available easy missions (not yet assigned to any team)
3. Get available hard missions (not yet assigned to any team)
4. Assign:
   - 2 random easy missions from available pool
   - 1 random hard mission from available pool
5. **If pool exhausted**: Allow mission duplicates (pick from entire pool)
6. Add to `state.missions[teamName]`
7. Sync to Firebase: `patchState({ missions: state.missions })`

### Uniqueness Guarantee
- Early teams get unique missions
- If all 13 easy missions are used and a 14th team joins, team 14 can receive a duplicate easy mission
- Same logic applies to hard missions

---

## Game Screen UI

### Tab Placement
New "🎯 Missions" tab appears after the "⚙️ Adjustments" tab in the game screen tab bar.

### Visibility Rules
Tab is visible **only if**:
- `secretMissionsEnabled === true` AND
- Player is in a team (`local.myTeamName !== ''`)

Non-team members (referees) never see the tab.

### Tab Content

```
🎯 Secret Missions

Easy Missions
☐ [Mission text 1]
☐ [Mission text 2]

Hard Missions
☐ [Mission text 3]
```

### Components

**Mission Row**:
- Mission text (left-aligned)
- Checkbox (right-aligned, clickable)
- No styling distinction between missions except the section headers

**Local Tracking**:
- Checkbox state stored in localStorage as `{teamName}_missions_checked`
- Checked state persists across page reloads
- No Firebase sync—purely local tracking
- Each team leader manages their own checkboxes independently

---

## Functions Required

- `renderMissionsTab()` - Render the missions tab UI
- `toggleMissionCheckbox(teamName, missionText)` - Toggle checkbox and update localStorage
- `assignMissionsToTeam(teamName)` - Assign 2 easy + 1 hard missions to a team when joining
- `getMissionPool()` - Return the hardcoded mission pool (easy and hard arrays)

---

## Integration Points

### Setup Screen
- Add toggle for secret missions below reward setup
- Update `renderOrgRewardSetup()` area

### Game Joining
- Call `assignMissionsToTeam()` in `joinGame()` after team is created

### Game Screen
- Add "🎯 Missions" tab after Adjustments tab
- Add tab handler in `switchTab()` function
- Render missions on tab switch

### Firebase Sync
- `missions` field synced with state via existing `patchState()` pattern
- No special handling needed

---

## Error Handling

- If `secretMissionsEnabled === true` but `state.missions[teamName]` is undefined, assign missions immediately
- If mission pool is corrupted or missing, log error and skip assignment
- If localStorage is unavailable, checkboxes still work but don't persist

---

## Testing Considerations

- Verify no two teams receive the same mission
- Verify missions are assigned when team joins
- Verify missions tab only appears when feature is enabled
- Verify checkbox state persists across reload
- Verify non-team players don't see tab
- Verify duplicates allowed when pool exhausted
- Test with 20+ teams to verify duplicate logic

---

## Implementation Notes

- Mission pool is hardcoded as two constant arrays in JavaScript
- Use existing `patchState()` pattern for Firebase syncs
- localStorage key format: `{teamName}_missions_checked`
- Checkbox state is array of mission texts (not indices)
- Tab visibility uses same pattern as teams tab (conditional display)
