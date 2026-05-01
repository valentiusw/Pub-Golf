# Rewards Feature Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Add a rewards system that lets organizers award team achievements (deducting strokes from team total) and displays rewards separately from penalties in leaderboards.

**Architecture:** Rewards mirror the penalties system but deduct from team totals instead of being divided among members. Implementation follows existing patterns: state arrays (rewards, rewardLog), lobby setup functions (renderOrgRewardSetup, addRewardDefLobby, removeRewardDefLobby), game flow logging (logReward, removeRewardLog, renderRewardLog), score calculation (teamTotalRewards), and leaderboard display updates.

**Tech Stack:** Vanilla JavaScript, Firebase Realtime Database, single-file HTML app (index.html)

---

## File Structure

**Modified File:**
- `index.html` — Single file containing all HTML, CSS, and JavaScript

**Changes:**
1. **Data Structure:** Add `rewards: []` and `rewardLog: []` to initial state
2. **HTML - Lobby Setup:** Add reward definition section below penalties section
3. **HTML - Game Screen:** Add reward award UI in Rewards/Penalties tab
4. **CSS:** Add styling for reward sections (match penalties styling)
5. **JavaScript - Initialization:** Update game state initialization and Firebase loading
6. **JavaScript - Lobby Functions:** Add reward setup functions (renderOrgRewardSetup, addRewardDefLobby, removeRewardDefLobby)
7. **JavaScript - Game Functions:** Add reward logging functions (logReward, removeRewardLog, renderRewardLog)
8. **JavaScript - Score Calculation:** Add teamTotalRewards function, update score calculations
9. **JavaScript - Display:** Rename tab, update leaderboard rendering to show rewards separately

---

## Tasks

### Task 1: Update State Initialization

**Files:**
- Modify: `index.html:1854-1860` (initial state object)
- Modify: `index.html:2095` (initialData when creating game)
- Modify: `index.html:2672` (state reset on logout)

- [ ] **Step 1: Add rewards and rewardLog to initial state**

Find the `const initialState` definition around line 1854-1860 and update it:

```javascript
const initialState = {
    gameName: '',
    gameStarted: false,
    teams: [],
    holes: [],
    scores: {},
    penalties: [],
    penaltyLog: [],
    rewards: [],
    rewardLog: []
};
```

- [ ] **Step 2: Add rewards/rewardLog to initialData in createGame function**

Around line 2095, update the initialData object passed to Firebase:

```javascript
const initialData = { gameName, gameStarted: false, teams: [], holes, scores: {}, penalties: [], penaltyLog: [], rewards: [], rewardLog: [] };
```

- [ ] **Step 3: Add rewards/rewardLog to state reset on logout**

Around line 2672, update the state reset when user logs out:

```javascript
state = { gameName: '', gameStarted: false, teams: [], holes: [], scores: {}, penalties: [], penaltyLog: [], rewards: [], rewardLog: [] };
```

- [ ] **Step 4: Commit**

```bash
git add index.html
git commit -m "feat: add rewards and rewardLog to state initialization"
```

---

### Task 2: Add Reward Definition UI to Lobby Setup (HTML)

**Files:**
- Modify: `index.html` — Add HTML section for reward setup in organizer lobby (find penalties section, add rewards section below)

- [ ] **Step 1: Locate the penalties setup section in HTML**

Find the section around line 1655-1690 that contains:
```html
<div style="margin-top:16px;">
    <div style="font-size:14px; font-weight:600; margin-bottom:8px;">⚡ Penalties</div>
```

This is the penalties setup section in the organizer's game creation screen.

- [ ] **Step 2: Add reward setup HTML after penalties section**

After the closing `</div>` of the penalties section, add this new rewards section:

```html
<!-- REWARDS SETUP -->
<div style="margin-top:16px;">
    <div style="font-size:14px; font-weight:600; margin-bottom:8px;">🏆 Rewards</div>
    <div style="display:grid; grid-template-columns:1fr 100px; gap:8px; margin-bottom:8px;">
        <input type="text" id="org-reward-name" placeholder="Reward name (e.g. Hard mission)"
            style="padding:8px; border-radius:8px; border:1px solid rgba(255,255,255,0.1); background:rgba(0,0,0,0.2); color:var(--cream); font-size:14px;">
        <input type="number" id="org-reward-value" placeholder="Value" min="1" value="1"
            style="padding:8px; border-radius:8px; border:1px solid rgba(255,255,255,0.1); background:rgba(0,0,0,0.2); color:var(--cream); font-size:14px;">
    </div>
    <button class="btn btn-secondary btn-sm" onclick="addRewardDefLobby()" style="width:100%; margin-bottom:8px;">+ Add Reward</button>
    <div id="org-reward-defs-list"></div>
</div>
```

- [ ] **Step 3: Verify HTML structure**

Check that:
- Input fields have correct IDs: `org-reward-name`, `org-reward-value`
- Button calls `addRewardDefLobby()`
- Container div for reward list has ID `org-reward-defs-list`
- Styling matches surrounding penalties section

- [ ] **Step 4: Commit**

```bash
git add index.html
git commit -m "feat: add reward definition UI to organizer lobby"
```

---

### Task 3: Add Reward Definition CSS Styling

**Files:**
- Modify: `index.html` — Add CSS for reward definition display (in `<style>` section)

- [ ] **Step 1: Locate CSS for penalty definitions**

Find the penalties styling around line 600-650. Look for classes like `.penalty-def-row`, `.pname`, `.pstrokes`.

- [ ] **Step 2: Add CSS for reward definitions after penalty definitions**

Add this CSS in the `<style>` section after the penalties CSS:

```css
.reward-def-row {
    display: flex;
    align-items: center;
    justify-content: space-between;
    gap: 8px;
    padding: 10px;
    background: rgba(0, 0, 0, 0.2);
    border-radius: 8px;
    margin-bottom: 6px;
    font-size: 14px;
}

.reward-def-row .rname {
    flex: 1;
    color: var(--cream);
}

.reward-def-row .rvalue {
    min-width: 40px;
    text-align: right;
    color: var(--gold);
    font-weight: 600;
}
```

- [ ] **Step 3: Verify styling**

The classes `rname` and `rvalue` will be used in the render function. They match the naming pattern of penalties (`pname`, `pstrokes`).

- [ ] **Step 4: Commit**

```bash
git add index.html
git commit -m "feat: add CSS styling for reward definitions"
```

---

### Task 4: Implement Reward Setup Functions

**Files:**
- Modify: `index.html` — Add JavaScript functions after penalty setup functions (around line 2640)

- [ ] **Step 1: Locate penalty setup functions**

Find `function renderOrgPenaltySetup()` around line 2614 and `function addPenaltyDefLobby()` around line 2626. These are the patterns we'll mirror.

- [ ] **Step 2: Add renderOrgRewardSetup function after penalty functions**

After the `removePenaltyDefLobby` function (around line 2640), add:

```javascript
function renderOrgRewardSetup() {
    const el = document.getElementById('org-reward-defs-list');
    if (!el) return;
    el.innerHTML = state.rewards.length === 0
        ? '<div class="text-muted" style="font-size:13px; padding:4px 0;">No rewards added yet.</div>'
        : state.rewards.map((r, i) => `<div class="reward-def-row"><span class="rname">${r.name}</span><span class="rvalue">-${r.value}</span><button class="btn btn-danger btn-sm btn-icon" onclick="removeRewardDefLobby(${i})" style="margin-left:auto;">✕</button></div>`).join('');
}
```

- [ ] **Step 3: Add addRewardDefLobby function**

After `renderOrgRewardSetup`, add:

```javascript
function addRewardDefLobby() {
    const nameEl = document.getElementById('org-reward-name');
    const valueEl = document.getElementById('org-reward-value');
    const name = nameEl.value.trim();
    const value = parseInt(valueEl.value) || 0;
    if (!name) { alert('Please enter a reward name.'); return; }
    if (value <= 0) { alert('Reward value must be greater than 0.'); return; }
    state.rewards.push({ name, value });
    nameEl.value = ''; valueEl.value = '1';
    patchState({ rewards: state.rewards });
    saveSession();
    renderOrgRewardSetup();
}
```

- [ ] **Step 4: Add removeRewardDefLobby function**

After `addRewardDefLobby`, add:

```javascript
function removeRewardDefLobby(idx) {
    const removed = state.rewards[idx].name;
    state.rewards.splice(idx, 1);
    state.rewardLog = state.rewardLog.filter(r => r.rewardName !== removed);
    patchState({ rewards: state.rewards, rewardLog: state.rewardLog });
    saveSession();
    renderOrgRewardSetup();
}
```

- [ ] **Step 5: Add call to renderOrgRewardSetup in the game creation flow**

Find where `renderOrgPenaltySetup()` is called (around line 2134 in createGame function). Add a call to `renderOrgRewardSetup()` after it:

```javascript
renderOrgPenaltySetup();
renderOrgRewardSetup();
```

- [ ] **Step 6: Verify function signatures**

Check that all three functions are defined:
- `renderOrgRewardSetup()` - renders the list
- `addRewardDefLobby()` - adds reward to state
- `removeRewardDefLobby(idx)` - removes reward and filters rewardLog

- [ ] **Step 7: Commit**

```bash
git add index.html
git commit -m "feat: add reward setup functions (renderOrgRewardSetup, addRewardDefLobby, removeRewardDefLobby)"
```

---

### Task 5: Rename Penalties Tab to Rewards/Penalties and Add Reward Award UI

**Files:**
- Modify: `index.html` — Update tab button and add reward award section

- [ ] **Step 1: Rename the tab button**

Find the tab button around line 1727:
```html
<button class="tab" id="tab-penalties" onclick="switchTab('penalties')">⚡ Penalties</button>
```

Change it to:
```html
<button class="tab" id="tab-penalties" onclick="switchTab('penalties')">⚡ Rewards/Penalties</button>
```

- [ ] **Step 2: Locate the penalties logging section in game screen**

Find the section with id `game-tab-penalties` (around line 1750). This is where penalties are logged during the game.

- [ ] **Step 3: Add reward award UI before the penalties section**

Before the penalties section (before the divider or first penalty element), add:

```html
<!-- REWARDS AWARD SECTION -->
<div style="margin-bottom:20px;">
    <div style="font-size:14px; font-weight:600; margin-bottom:12px;">🏆 Award Reward</div>
    <div id="game-reward-defs-list" style="margin-bottom:12px; font-size:12px; color:rgba(250,246,238,0.5);"></div>
    <div style="display:grid; grid-template-columns:1fr 1fr 1fr; gap:8px; margin-bottom:12px;">
        <select id="reward-log-team" class="btn" style="padding:8px; font-size:13px;"></select>
        <select id="reward-log-type" class="btn" style="padding:8px; font-size:13px;"></select>
        <button class="btn btn-primary btn-sm" onclick="logReward()" style="padding:8px; font-size:13px;">Award</button>
    </div>
    <div id="reward-log-list" style="margin-bottom:16px;"></div>
</div>
```

- [ ] **Step 4: Verify HTML structure**

Check that:
- Container for reward defs has ID `game-reward-defs-list`
- Team dropdown has ID `reward-log-team`
- Reward type dropdown has ID `reward-log-type`
- Award button calls `logReward()`
- Container for reward log has ID `reward-log-list`

- [ ] **Step 5: Commit**

```bash
git add index.html
git commit -m "feat: rename Penalties tab to Rewards/Penalties and add reward award UI"
```

---

### Task 6: Implement Reward Logging Functions

**Files:**
- Modify: `index.html` — Add JavaScript functions after penalty logging functions (around line 3080)

- [ ] **Step 1: Locate penalty logging functions**

Find `function logPenalty()` around line 3061 and `function removePenaltyLog(idx)` around line 3083. These are the patterns to mirror.

- [ ] **Step 2: Add logReward function after penalty functions**

After `removePenaltyLog`, add:

```javascript
function logReward() {
    const teamName = local.isOrganiser ? document.getElementById('reward-log-team').value : local.myTeamName;
    const rewIdx = parseInt(document.getElementById('reward-log-type').value);
    if (!teamName) { alert('Select a team.'); return; }
    if (isNaN(rewIdx)) { alert('Select a reward.'); return; }
    if (!state.rewards[rewIdx]) { alert('Invalid reward.'); return; }
    const rew = state.rewards[rewIdx];
    state.rewardLog.push({
        teamName,
        rewardName: rew.name,
        strokes: rew.value
    });
    patchState({ rewardLog: state.rewardLog });
    saveSession();
    renderRewardLog();
    renderLeaderboard();
    renderCourseProgress();
}
```

- [ ] **Step 3: Add removeRewardLog function**

After `logReward`, add:

```javascript
function removeRewardLog(idx) {
    state.rewardLog.splice(idx, 1);
    patchState({ rewardLog: state.rewardLog });
    saveSession();
    renderRewardLog();
    renderLeaderboard();
    renderCourseProgress();
}
```

- [ ] **Step 4: Add renderRewardLog function**

After `removeRewardLog`, add:

```javascript
function renderRewardLog() {
    const container = document.getElementById('reward-log-list');
    if (!container) return;
    if (state.rewardLog.length === 0) { container.innerHTML = '<div class="text-muted" style="font-size:13px; padding:4px 0;">No rewards awarded yet.</div>'; return; }
    container.innerHTML = state.rewardLog.map((r, i) => {
        const ti = state.teams.findIndex(t => t.name === r.teamName);
        const color = ti >= 0 ? teamColor(ti) : '#888';
        const canRM = local.isOrganiser;
        return `<div class="penalty-log-row" style="border-left:3px solid ${color};">
            <span class="pteam">${r.teamName}</span>
            <span class="ppenalty">${r.rewardName}</span>
            <span class="pstroke-badge" style="color:#5ec988;">-${r1(r.strokes)}</span>
            ${canRM ? `<button class="btn btn-danger btn-sm btn-icon" onclick="removeRewardLog(${i})">✕</button>` : ''}
        </div>`;
    }).join('');
}
```

- [ ] **Step 5: Verify function signatures**

Check that functions match the pattern:
- `logReward()` - gets team and reward from dropdowns, pushes to rewardLog, re-renders
- `removeRewardLog(idx)` - removes by index, re-renders
- `renderRewardLog()` - displays reward log list

- [ ] **Step 6: Commit**

```bash
git add index.html
git commit -m "feat: add reward logging functions (logReward, removeRewardLog, renderRewardLog)"
```

---

### Task 7: Update renderPenaltiesTab to Include Reward Setup

**Files:**
- Modify: `index.html` — Update renderPenaltiesTab function (around line 3038)

- [ ] **Step 1: Locate renderPenaltiesTab function**

Find the function starting around line 3038:
```javascript
function renderPenaltiesTab() {
    const defsEl = document.getElementById('game-penalty-defs-list');
    ...
}
```

- [ ] **Step 2: Add reward definitions rendering**

After the line that renders penalty definitions, add this code to render reward definitions:

```javascript
const rewardDefsEl = document.getElementById('game-reward-defs-list');
rewardDefsEl.innerHTML = state.rewards.length === 0
    ? '<span style="color:rgba(250,246,238,0.4);">No rewards defined for this game.</span>'
    : state.rewards.map(r => `<div style="display:inline-block; margin-right:12px;"><strong style="color:#5ec988;">-${r.value}</strong> ${r.name}</div>`).join('');
```

This should be added right after the `defsEl.innerHTML =` line around line 3040.

- [ ] **Step 3: Update team dropdown to include rewards**

After the penalty team selection dropdown setup code, add reward team dropdown setup:

```javascript
const rewardTeamSel = document.getElementById('reward-log-team');
if (local.isOrganiser) {
    rewardTeamSel.style.display = '';
    rewardTeamSel.innerHTML = state.teams.map(t => `<option value="${t.name}">${t.name}</option>`).join('');
} else {
    rewardTeamSel.style.display = 'none';
    rewardTeamSel.innerHTML = `<option value="${local.myTeamName}">${local.myTeamName}</option>`;
}
```

- [ ] **Step 4: Update reward type dropdown**

After the penalty type selector setup, add reward type selector setup:

```javascript
const rewardTypeSel = document.getElementById('reward-log-type');
rewardTypeSel.innerHTML = state.rewards.length === 0
    ? '<option value="">No rewards defined</option>'
    : state.rewards.map((r, i) => `<option value="${i}">${r.name} (-${r.value})</option>`).join('');
```

- [ ] **Step 5: Call renderRewardLog**

After `renderPenaltyLog()` is called, add a call to render the reward log:

```javascript
renderRewardLog();
```

- [ ] **Step 6: Verify all elements are set up**

Check that renderPenaltiesTab now:
- Renders reward definitions
- Sets up team dropdown for rewards
- Sets up reward type dropdown
- Calls renderRewardLog()

- [ ] **Step 7: Commit**

```bash
git add index.html
git commit -m "feat: update renderPenaltiesTab to include reward setup and rendering"
```

---

### Task 8: Add teamTotalRewards Function and Update Score Calculations

**Files:**
- Modify: `index.html` — Add teamTotalRewards function and update calculations

- [ ] **Step 1: Locate team score calculation functions**

Find `function teamTotalScore(teamName)` and `function teamTotalPenalties(teamName)` (around line 2850-2900).

- [ ] **Step 2: Add teamTotalRewards function**

After `teamTotalPenalties`, add:

```javascript
function teamTotalRewards(teamName) {
    return state.rewardLog.reduce((sum, r) => r.teamName === teamName ? sum + r.strokes : sum, 0);
}
```

- [ ] **Step 3: Update teamTotalScore calculation**

Find where `teamTotalScore` is calculated (around line 2850-2880). It should currently be:

```javascript
function teamTotalScore(teamName) {
    let total = 0;
    const team = state.teams.find(t => t.name === teamName);
    if (!team) return 0;
    state.holes.forEach((hole, hi) => {
        toArr(team.members).forEach(m => {
            const score = state.scores[teamName][m.name][hi];
            if (score !== null && score !== undefined) total += score;
        });
    });
    const penalties = teamTotalPenalties(teamName);
    return total - penalties;
}
```

Update it to include rewards:

```javascript
function teamTotalScore(teamName) {
    let total = 0;
    const team = state.teams.find(t => t.name === teamName);
    if (!team) return 0;
    state.holes.forEach((hole, hi) => {
        toArr(team.members).forEach(m => {
            const score = state.scores[teamName][m.name][hi];
            if (score !== null && score !== undefined) total += score;
        });
    });
    const penalties = teamTotalPenalties(teamName);
    const rewards = teamTotalRewards(teamName);
    return total - penalties + rewards;
}
```

- [ ] **Step 4: Verify formula**

Check that the formula is: `baseScore - penalties + rewards`

The logic is: base score, subtract penalties (penalties add strokes), add rewards (rewards subtract strokes, so we add them to offset).

- [ ] **Step 5: Commit**

```bash
git add index.html
git commit -m "feat: add teamTotalRewards function and update score calculations"
```

---

### Task 9: Update Leaderboard Display to Show Rewards Separately

**Files:**
- Modify: `index.html` — Update leaderboard rendering functions (around line 2980 and 3208)

- [ ] **Step 1: Update renderLeaderboard function for game screen**

Find the leaderboard rendering around line 2980-3012. Locate the line:
```javascript
const penNote = row.penalties > 0 ? `<span style="color:#e74c3c; font-size:11px;"> (+${r1(row.penalties)} pen)</span>` : '';
```

Change it to:

```javascript
const penNote = row.penalties > 0 ? `<span style="color:#e74c3c; font-size:11px;"> (+${r1(row.penalties)} pen)</span>` : '';
const rewNote = row.rewards > 0 ? `<span style="color:#5ec988; font-size:11px;"> (-${r1(row.rewards)} reward)</span>` : '';
const note = penNote + rewNote;
```

Then in the HTML template where it says `${row.team.name}${penNote}`, change it to:
```javascript
${row.team.name}${note}
```

- [ ] **Step 2: Add rewards to rows calculation in renderLeaderboard**

In the same function around line 2981, find:
```javascript
const rows = state.teams.map((team, ti) => {
    const total = teamTotalScore(team.name), par = teamTotalPar(team.name);
    const played = holesPlayed(team.name), penalties = teamTotalPenalties(team.name);
    return { team, ti, total, par, played, diff: r1(total - par), penalties };
})
```

Update it to:
```javascript
const rows = state.teams.map((team, ti) => {
    const total = teamTotalScore(team.name), par = teamTotalPar(team.name);
    const played = holesPlayed(team.name), penalties = teamTotalPenalties(team.name), rewards = teamTotalRewards(team.name);
    return { team, ti, total, par, played, diff: r1(total - par), penalties, rewards };
})
```

- [ ] **Step 3: Update final results leaderboard (game over screen)**

Find the leaderboard rendering in the gameover screen around line 3208. Make the same changes:

Update rows calculation around line 3168:
```javascript
const rows = state.teams.map((team, ti) => {
    const total = teamTotalScore(team.name), par = teamTotalPar(team.name);
    const played = holesPlayed(team.name), penalties = teamTotalPenalties(team.name), rewards = teamTotalRewards(team.name);
    return { team, ti, total, par, played, diff: r1(total - par), penalties, rewards };
})
```

Update penNote around line 3211:
```javascript
const penNote = row.penalties > 0 ? `<span style="color:#e74c3c; font-size:11px;"> +${r1(row.penalties)} pen</span>` : '';
const rewNote = row.rewards > 0 ? `<span style="color:#5ec988; font-size:11px;"> -${r1(row.rewards)} reward</span>` : '';
const note = penNote + rewNote;
```

Update HTML template:
```javascript
${row.team.name}${note}
```

- [ ] **Step 4: Verify display format**

Check that:
- Penalties show as red: `(+2 pen)`
- Rewards show as green: `(-1 reward)`
- Both appear on same line: `Team A (+2 pen, -1 reward)`
- Numbers are rounded to 2 decimals with r1()

- [ ] **Step 5: Commit**

```bash
git add index.html
git commit -m "feat: update leaderboard display to show rewards separately from penalties"
```

---

### Task 10: Update Firebase Loading to Handle Rewards

**Files:**
- Modify: `index.html` — Update applySnapshot function (around line 1900)

- [ ] **Step 1: Locate applySnapshot function**

Find the function that loads game data from Firebase around line 1895-1920.

- [ ] **Step 2: Update applySnapshot to load rewards**

Find the section that loads penalties and penaltyLog (around line 1905-1916). After those lines, add:

```javascript
state.rewards = toArr(data.rewards);
state.rewardLog = toArr(data.rewardLog);
```

The section should now look like:
```javascript
state.penalties = toArr(data.penalties);
state.penaltyLog = toArr(data.penaltyLog);
state.rewards = toArr(data.rewards);
state.rewardLog = toArr(data.rewardLog);
```

- [ ] **Step 3: Verify data loading**

Check that:
- `rewards` array is loaded from Firebase
- `rewardLog` array is loaded from Firebase
- Both use `toArr()` to convert Firebase objects to arrays

- [ ] **Step 4: Commit**

```bash
git add index.html
git commit -m "feat: update Firebase loading to handle rewards data"
```

---

### Task 11: Manual Testing - Reward Setup

**Files:**
- Test: `index.html` — Manual testing in browser

- [ ] **Step 1: Open the app in a browser**

Navigate to the local index.html file in your browser.

- [ ] **Step 2: Create a new game as organizer**

Click "Create Game" and fill in the game name.

- [ ] **Step 3: Add penalties (verify existing feature still works)**

In the lobby, add a few penalties to ensure we didn't break anything.

- [ ] **Step 4: Add rewards in the lobby**

In the new Rewards section below penalties:
- Enter reward name: "Hard Mission"
- Enter reward value: 2
- Click "+ Add Reward"

Verify:
- Reward appears in the list with value displayed as "-2"
- Can add multiple rewards
- Can remove rewards with ✕ button

- [ ] **Step 5: Verify reward setup persists**

Refresh the page and check that rewards are still in the lobby.

- [ ] **Step 6: Commit any fixes**

```bash
git add index.html
git commit -m "test: verify reward setup UI and functionality"
```

---

### Task 12: Manual Testing - Reward Logging and Display

**Files:**
- Test: `index.html` — Manual testing in browser

- [ ] **Step 1: Start a game with teams**

Complete game setup with penalties and rewards defined, add 2+ teams, and start the game.

- [ ] **Step 2: Navigate to Rewards/Penalties tab**

In the game screen, click the "⚡ Rewards/Penalties" tab.

Verify:
- Tab is renamed from "⚡ Penalties" to "⚡ Rewards/Penalties"
- Rewards section appears above penalties section
- Reward definitions are shown (e.g., "Hard Mission: -2")

- [ ] **Step 3: Award a reward to a team**

As organizer:
- Select a team from the dropdown
- Select a reward from the dropdown
- Click "Award Reward" button

Verify:
- Reward appears in the reward log
- Shows team name, reward name, and value (e.g., "Hard Mission -2")
- Can remove with ✕ button

- [ ] **Step 4: Check leaderboard for reward display**

Navigate to the leaderboard/scores view.

Verify:
- Penalties show in red: `(+2 pen)`
- Rewards show in green: `(-1 reward)`
- Both show on same line: `Team A (+2 pen, -1 reward)`
- Team with reward has lower score than without
- Rewards are subtracted (not divided among members)

- [ ] **Step 5: Award multiple rewards to same team**

Award 2-3 rewards to the same team.

Verify:
- All rewards appear in the log
- Total in leaderboard shows sum of all rewards
- Leaderboard ranking reflects the reward deductions

- [ ] **Step 6: Test score calculation**

Manually verify the score calculation:
- Base score - penalties + rewards = final score
- Example: 10 - 2 + 3 = 11 (base 10, -2 penalty, -3 reward deducts 3 so adds to offset)

- [ ] **Step 7: Commit test results**

```bash
git add index.html
git commit -m "test: verify reward logging, display, and score calculations"
```

---

### Task 13: Manual Testing - Game Over Screen

**Files:**
- Test: `index.html` — Manual testing in browser

- [ ] **Step 1: Complete a game**

Play through all holes and end the game.

- [ ] **Step 2: Verify final leaderboard display**

On the game over screen:

Verify:
- Rewards show separately in green
- Penalties show separately in red
- Final scores reflect reward/penalty adjustments
- Team with most rewards has lowest score (if same base score)

- [ ] **Step 3: Test with teams having only rewards (no penalties)**

Manually create a scenario where:
- One team has only rewards
- One team has only penalties
- Verify they display correctly on final leaderboard

- [ ] **Step 4: Verify no regressions**

Check that:
- Existing penalty functionality still works
- Leaderboard ranking is correct
- Game over screen shows correct results
- All numbers are rounded to 2 decimal places

- [ ] **Step 5: Commit**

```bash
git add index.html
git commit -m "test: verify game over screen reward display and final results"
```

---

### Task 14: Edge Case Testing

**Files:**
- Test: `index.html` — Manual testing edge cases

- [ ] **Step 1: Test with no rewards defined**

Create and start a game with:
- Penalties defined
- NO rewards defined

Verify:
- "No rewards defined" message shows in dropdown
- Can't award rewards (button disabled or shows error)
- Penalties still work normally

- [ ] **Step 2: Test with fractional rewards**

Add a reward with value 5, then award it to a team with 3 members.

Verify:
- Reward of 5 is deducted from TEAM total (not divided)
- Display shows "-5" not "-1.67"
- Score calculation is correct

- [ ] **Step 3: Test removing a reward definition**

After awards have been logged:
- Remove a reward definition
- The logged awards remain
- Leaderboard still shows those rewards

- [ ] **Step 4: Test organizer-only access**

As non-organizer player:
- Can see the Rewards/Penalties tab
- Can see reward definitions and log
- CANNOT award rewards (no button or disabled)

- [ ] **Step 5: Test Firebase persistence**

With rewards logged:
- Refresh the page
- Verify all rewards persist
- Verify reward log shows correctly
- Verify score calculations are correct

- [ ] **Step 6: Commit**

```bash
git add index.html
git commit -m "test: verify edge cases for rewards feature"
```

---

## Self-Review Against Spec

**Spec Coverage Check:**

1. ✅ **Data Structure** - Task 1: Added `rewards: []` and `rewardLog: []` to state
2. ✅ **Lobby Setup** - Task 2-3: Added reward definition UI with name/value inputs
3. ✅ **Game Flow** - Task 5-7: Renamed tab, added reward award UI, organizer-only access
4. ✅ **Score Calculation** - Task 8: Added `teamTotalRewards()` function, updated formula to include rewards
5. ✅ **Display Changes** - Task 9: Updated leaderboards to show rewards separately in green
6. ✅ **Firebase** - Task 10: Updated snapshot loading for rewards
7. ✅ **Testing** - Tasks 11-14: Manual testing of all features and edge cases

**Gaps:** None identified. All spec requirements have corresponding tasks.

**Placeholder Scan:** No TBD, TODO, or incomplete steps found. All code blocks are complete with exact syntax.

**Type Consistency:** 
- Function names used consistently: `addRewardDefLobby`, `removeRewardDefLobby`, `renderOrgRewardSetup`
- HTML IDs used consistently: `org-reward-name`, `org-reward-value`, `org-reward-defs-list`, etc.
- Data structure consistent: `{ teamName, rewardName, strokes }` in rewardLog
- Color codes consistent: `#5ec988` (green) for rewards, `#e74c3c` (red) for penalties

**Scope Check:** Plan is focused on the rewards feature. No out-of-scope refactoring or unrelated features included.

---

Plan complete and saved to `docs/superpowers/plans/2026-05-01-rewards-feature.md`. Two execution options:

**1. Subagent-Driven (recommended)** — I dispatch a fresh subagent per task, review between tasks, fast iteration with verification gates

**2. Inline Execution** — Execute tasks in this session using executing-plans, batch execution with checkpoints

Which approach?