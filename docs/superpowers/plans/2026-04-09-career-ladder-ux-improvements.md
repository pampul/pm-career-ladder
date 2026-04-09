# PM Career Ladder UX Improvements — Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Reduce assessment length from 20 to 14 dimensions by removing Potential Misalignments from the eval flow, add a focus-level selector to reduce visual noise per dimension, and display all 6 anti-pattern cards read-only in the results.

**Architecture:** Single `index.html` file with inline CSS and JS. Data is in `CAREER_LADDER` array (20 dimensions across 5 categories). The plan introduces `EVAL_DIMS` (14 dims, categories 0–3) and `MISALIGNMENT_DIMS` (6 dims, category 4), a `focusLevel` state field, and reworks the results section.

**Tech Stack:** Vanilla HTML/CSS/JS, no build step. Edit `index.html` only.

---

## Files

- **Modify:** `index.html` — all changes are in this single file

---

## Task 1: Separate EVAL_DIMS from MISALIGNMENT_DIMS

Remove the 6 Potential Misalignment dimensions from the evaluation flow. Update navigation, stepper, progress bar, and scoring to operate on 14 dimensions only.

**Files:**
- Modify: `index.html` — JS constants, navigation functions, computeResults, renderStepper, progress bar HTML

- [ ] **Step 1: Add EVAL_DIMS and MISALIGNMENT_DIMS constants after ALL_DIMS (line ~644)**

Find this code:
```javascript
const ALL_DIMS = getAllDimensions();
const TOTAL_DIMS = ALL_DIMS.length;
```

Replace with:
```javascript
const ALL_DIMS = getAllDimensions();
const EVAL_DIMS = ALL_DIMS.filter(d => d.catIdx < 4);
const MISALIGNMENT_DIMS = ALL_DIMS.filter(d => d.catIdx === 4);
const TOTAL_DIMS = EVAL_DIMS.length; // 14
```

- [ ] **Step 2: Open the file in a browser and verify the console has no errors**

Open `index.html` directly in a browser (file:// is fine). Open DevTools console. Expected: no errors.

- [ ] **Step 3: Update renderStepper() to use only 4 categories**

Find the `renderStepper()` function. It starts with:
```javascript
function renderStepper() {
  const shortNames = ["Impact", "Craft", "People", "Strategy", "Misalign."];
  const currentDim = ALL_DIMS[state.currentDimIndex];
  let html = '<div class="stepper">';
  CAREER_LADDER.forEach((cat, i) => {
    if (i > 0) html += '<div class="step-line"></div>';
    const isActive = i === currentDim.catIdx;
    const catDims = cat.dimensions;
    const allDone = catDims.every((_, di) => state.selections.hasOwnProperty(`${i}-${di}`));
    const cls = allDone ? 'completed' : (isActive ? 'active' : '');
    // Click to go to first dim of that category
    const firstGlobal = ALL_DIMS.findIndex(d => d.catIdx === i);
```

Replace the entire `renderStepper()` function with:
```javascript
function renderStepper() {
  const shortNames = ["Impact", "Craft", "People", "Strategy"];
  const currentDim = EVAL_DIMS[state.currentDimIndex];
  let html = '<div class="stepper">';
  CAREER_LADDER.slice(0, 4).forEach((cat, i) => {
    if (i > 0) html += '<div class="step-line"></div>';
    const isActive = i === currentDim.catIdx;
    const catDims = cat.dimensions;
    const allDone = catDims.every((_, di) => state.selections.hasOwnProperty(`${i}-${di}`));
    const cls = allDone ? 'completed' : (isActive ? 'active' : '');
    const firstGlobal = EVAL_DIMS.findIndex(d => d.catIdx === i);
    html += `<button type="button" class="step-col" onclick="goToDimension(${firstGlobal})" aria-label="${cat.category}${allDone ? ', completed' : ''}" ${isActive ? 'aria-current="step"' : ''}>
      <div class="step-circle ${cls}">${allDone ? '<svg width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="3" stroke-linecap="round" stroke-linejoin="round"><polyline points="20 6 9 17 4 12"/></svg>' : (i + 1)}</div>
      <div class="step-label ${isActive ? 'active' : ''}">${shortNames[i]}</div>
    </button>`;
  });
  html += '</div>';
  document.getElementById('eval-stepper').innerHTML = html;
}
```

- [ ] **Step 4: Update renderEvaluation() to use EVAL_DIMS**

Find this line inside `renderEvaluation()`:
```javascript
  const dim = ALL_DIMS[state.currentDimIndex];
```

Replace with:
```javascript
  const dim = EVAL_DIMS[state.currentDimIndex];
```

- [ ] **Step 5: Update progress bar HTML text**

Find in the HTML (around line 292):
```html
<span class="progress-bar-text"><strong id="progress-count">0</strong> of 20 dimensions &mdash; <span id="progress-percent">0%</span></span>
```

Replace with:
```html
<span class="progress-bar-text"><strong id="progress-count">0</strong> of 14 dimensions &mdash; <span id="progress-percent">0%</span></span>
```

- [ ] **Step 6: Update hero meta text**

Find:
```html
<p class="meta">20 dimensions &middot; 5 levels &middot; ~25 min</p>
```

Replace with:
```html
<p class="meta">14 dimensions &middot; 5 levels &middot; ~20 min</p>
```

- [ ] **Step 7: Update computeResults() to use EVAL_DIMS only**

Find the entire `computeResults()` function (starts at `function computeResults() {`). Replace it with:

```javascript
function computeResults() {
  const dimensionScores = [];

  EVAL_DIMS.forEach(dim => {
    const key = `${dim.catIdx}-${dim.dimIdx}`;
    const sel = state.selections[key];
    const skipped = !state.selections.hasOwnProperty(key) || sel === null;
    dimensionScores.push({
      category: dim.category,
      categoryIdx: dim.catIdx,
      dimension: dim.name,
      level: skipped ? null : sel + 1,
      levelIdx: skipped ? null : sel,
      levelName: skipped ? null : LEVELS[sel],
      skipped
    });
  });

  const categoryAverages = SCORABLE_CATEGORIES.map(catIdx => {
    const dims = dimensionScores.filter(d => d.categoryIdx === catIdx && !d.skipped);
    if (dims.length === 0) return null;
    return dims.reduce((sum, d) => sum + d.level, 0) / dims.length;
  });

  const evaluatedDims = dimensionScores.filter(d => !d.skipped);
  const overallAvg = evaluatedDims.length > 0
    ? evaluatedDims.reduce((sum, d) => sum + d.level, 0) / evaluatedDims.length
    : 0;
  const overallLevel = Math.round(overallAvg);
  const overallLevelName = overallLevel > 0 ? LEVELS[overallLevel - 1] : 'N/A';

  const scored = evaluatedDims.map(d => ({ ...d, delta: d.level - overallAvg }));
  const strengthsSorted = [...scored].sort((a, b) => b.delta - a.delta || a.categoryIdx - b.categoryIdx);
  const growthSorted = [...scored].sort((a, b) => a.delta - b.delta || a.categoryIdx - b.categoryIdx);

  return { categoryAverages, dimensionScores, overallLevel, overallLevelName, overallAvg, strengths: strengthsSorted.slice(0, 3), growthAreas: growthSorted.slice(0, 3) };
}
```

- [ ] **Step 8: Update renderResults() — remove old misalignment rendering, update detailed table**

Find inside `renderResults()` the entire misalignment block:
```javascript
  // Misalignments
  const mList = document.getElementById('misalignments-list');
  const evalMisalignments = results.misalignments.filter(m => !m.skipped);
  if (evalMisalignments.length === 0) {
    mList.innerHTML = '<p style="color:var(--gray-400)">No misalignment dimensions were evaluated.</p>';
  } else {
    mList.innerHTML = evalMisalignments.map(m => {
      const dim = CAREER_LADDER[4].dimensions.find(d => d.name === m.dimension);
      const data = dim.levels[m.levelName];
      return `<div class="misalignment-item">
        <div class="dim-name">${m.dimension}</div>
        <div class="pattern-level">${m.levelName} pattern</div>
        <div class="pattern">${data.core}${data.pm ? '<br><br><span style="color:var(--gray-500);font-style:italic">[PM]' + data.pm + '</span>' : ''}</div>
      </div>`;
    }).join('');
  }
```

Replace with:
```javascript
  // Points de vigilance — placeholder content, full rendering in Task 3
  document.getElementById('misalignments-list').innerHTML = '';
```

Find the detailed table block inside `renderResults()`:
```javascript
  let thtml = `<thead><tr><th>Category</th><th>Dimension</th><th>Level</th>${hasGap ? '<th>Target</th><th>Gap</th>' : ''}</tr></thead><tbody>`;
  const allScores = [...results.dimensionScores, ...results.misalignments];
  allScores.forEach(s => {
```

Replace the `allScores` line only:
```javascript
  let thtml = `<thead><tr><th>Category</th><th>Dimension</th><th>Level</th>${hasGap ? '<th>Target</th><th>Gap</th>' : ''}</tr></thead><tbody>`;
  results.dimensionScores.forEach(s => {
```

- [ ] **Step 9: Update generateReport() — remove misalignment section, fix detailed breakdown**

Find:
```javascript
  md += `\n## Potential Misalignments\n\n| Dimension | Selected Pattern |\n|-----------|------------------|\n`;
  results.misalignments.filter(m => !m.skipped).forEach(m => { const d = CAREER_LADDER[4].dimensions.find(x => x.name === m.dimension); md += `| ${m.dimension} | ${m.levelName}: ${d.levels[m.levelName].core} |\n`; });
```

Replace with:
```javascript
  // Potential Misalignments are no longer evaluated — omitted from report
```

Find in `generateReport()`:
```javascript
  md += `\n## Detailed Breakdown\n\n`;
  CAREER_LADDER.forEach((cat, catIdx) => {
```

Replace with:
```javascript
  md += `\n## Detailed Breakdown\n\n`;
  CAREER_LADDER.slice(0, 4).forEach((cat, catIdx) => {
```

- [ ] **Step 10: Update generateAIPack() — use only dimensionScores**

Find:
```javascript
  text += `### Dimension-by-Dimension Results\n\n| Category | Dimension | Selected Level | Supporting Example |\n|----------|-----------|----------------|--------------------|\n`;
  [...results.dimensionScores, ...results.misalignments].forEach(s => {
```

Replace:
```javascript
  text += `### Dimension-by-Dimension Results\n\n| Category | Dimension | Selected Level | Supporting Example |\n|----------|-----------|----------------|--------------------|\n`;
  results.dimensionScores.forEach(s => {
```

- [ ] **Step 11: Verify in browser**

Open `index.html`. Start an assessment, complete a mode selection and setup. Confirm the stepper shows 4 steps (not 5). Confirm the progress bar shows "of 14 dimensions". Navigate to the last dimension and confirm clicking Next leads to context questions. Go to results and confirm no JS errors in console.

- [ ] **Step 12: Commit**

```bash
git add index.html
git commit -m "feat: remove Potential Misalignments from eval flow (14 dims)"
```

---

## Task 2: Add focus level to setup + focused level display

Add a "current or target level" selector for IC mode. On each dimension, show only the focus level ± 1 by default, with a toggle to expand to all 5.

**Files:**
- Modify: `index.html` — CSS (new classes), state object, renderSetup, startEvaluation, new helper functions, renderEvaluation

- [ ] **Step 1: Add focusLevel to state object**

Find:
```javascript
  mode: null,
  name: "",
  currentLevel: null,
  targetLevel: null,
  currentDimIndex: 0,
```

Replace with:
```javascript
  mode: null,
  name: "",
  currentLevel: null,
  targetLevel: null,
  focusLevel: null,
  currentDimIndex: 0,
```

- [ ] **Step 2: Add CSS for hidden levels and toggle button**

Find the end of the CSS `</style>` tag (just before `</head>`). Add before it:

```css
/* Focus level filtering */
.level-card.level-hidden{display:none}
#eval-content.show-all .level-card.level-hidden{display:block}
.show-all-btn{display:block;margin:10px auto 0;padding:8px 16px;min-height:44px;background:none;border:none;color:var(--gray-400);font-size:.8rem;cursor:pointer;text-decoration:underline;font:inherit;border-radius:var(--radius);transition:var(--transition)}
.show-all-btn:hover{color:var(--gray-600);background:var(--gray-50)}
.show-all-btn:focus-visible{outline:2px solid var(--la-focus);outline-offset:2px}
```

- [ ] **Step 3: Add focus level selector to IC mode in renderSetup()**

Find inside `renderSetup()`:
```javascript
  let html = `
    <div class="form-group">
      <label for="input-name">${isManager ? "Your report's name" : "Your name"}</label>
      <input type="text" id="input-name" placeholder="Enter name" value="${state.name}">
    </div>
  `;
```

Replace with:
```javascript
  let html = `
    <div class="form-group">
      <label for="input-name">${isManager ? "Your report's name" : "Your name"}</label>
      <input type="text" id="input-name" placeholder="Enter name" value="${state.name}">
    </div>
  `;

  if (!isManager) {
    html += `
      <div class="form-group">
        <label for="select-focus-level">Your current or target level</label>
        <select id="select-focus-level">
          <option value="">Select a level</option>
          ${LEVELS.map(l => `<option value="${l}" ${state.focusLevel === l ? 'selected' : ''}>${l}</option>`).join('')}
        </select>
      </div>
    `;
  }
```

- [ ] **Step 4: Capture focusLevel in startEvaluation()**

Find inside `startEvaluation()`, after the name validation block:
```javascript
  state.name = name;

  if (state.mode === 'manager') {
    const currentSel = document.getElementById('select-current-level');
    const targetSel = document.getElementById('select-target-level');
    if (currentSel && document.getElementById('current-level-field').style.display !== 'none') {
      state.currentLevel = currentSel.value;
    }
    if (targetSel && document.getElementById('target-level-field').style.display !== 'none') {
      state.targetLevel = targetSel.value;
    }
  }
```

Replace with:
```javascript
  state.name = name;

  if (state.mode === 'manager') {
    const currentSel = document.getElementById('select-current-level');
    const targetSel = document.getElementById('select-target-level');
    if (currentSel && document.getElementById('current-level-field').style.display !== 'none') {
      state.currentLevel = currentSel.value;
    }
    if (targetSel && document.getElementById('target-level-field').style.display !== 'none') {
      state.targetLevel = targetSel.value;
    }
    state.focusLevel = state.targetLevel || state.currentLevel || null;
  } else {
    const focusSel = document.getElementById('select-focus-level');
    if (!focusSel.value) {
      focusSel.style.borderColor = 'var(--red-500)';
      focusSel.focus();
      return;
    }
    state.focusLevel = focusSel.value;
  }
```

- [ ] **Step 5: Add getVisibleLevelRange() and toggleAllLevels() helper functions**

Add these two functions right before the `// EVALUATION — WIZARD MODE` comment:

```javascript
function getVisibleLevelRange() {
  if (!state.focusLevel) return null; // null = show all
  const idx = LEVELS.indexOf(state.focusLevel);
  return { start: Math.max(0, idx - 1), end: Math.min(4, idx + 1) };
}

function toggleAllLevels(btn) {
  const content = document.getElementById('eval-content');
  content.classList.toggle('show-all');
  btn.textContent = content.classList.contains('show-all') ? 'Masquer les niveaux extrêmes' : 'Voir tous les niveaux';
}
```

- [ ] **Step 6: Apply level-hidden class in renderEvaluation() and add toggle button**

Inside `renderEvaluation()`, find the LEVELS.forEach loop that renders level cards:
```javascript
  {
    LEVELS.forEach((level, levelIdx) => {
      const data = dim.levels[level];
      const isSelected = sel === levelIdx;
      const isExpanded = isSelected;
      const preview = data.core.split('.')[0] + '.';

      html += `<div class="level-card ${isSelected ? 'selected' : ''} ${isExpanded ? 'expanded' : ''}" data-level="${levelIdx}" aria-expanded="${isExpanded}" onclick="toggleCard(${levelIdx})">
```

Replace with:
```javascript
  {
    const range = getVisibleLevelRange();
    LEVELS.forEach((level, levelIdx) => {
      const data = dim.levels[level];
      const isSelected = sel === levelIdx;
      const isExpanded = isSelected;
      const preview = data.core.split('.')[0] + '.';
      const isHidden = range !== null && (levelIdx < range.start || levelIdx > range.end);

      html += `<div class="level-card ${isSelected ? 'selected' : ''} ${isExpanded ? 'expanded' : ''} ${isHidden ? 'level-hidden' : ''}" data-level="${levelIdx}" aria-expanded="${isExpanded}" onclick="toggleCard(${levelIdx})">
```

Then find the closing brace of the LEVELS.forEach block and the closing brace of the surrounding block — right after the last `});` that closes `LEVELS.forEach`. Add the toggle button before the closing of the outer block:

Find:
```javascript
    });
  }

  document.getElementById('eval-content').innerHTML = html;
```

Replace with:
```javascript
    });

    if (range !== null && (range.start > 0 || range.end < 4)) {
      html += `<button type="button" class="show-all-btn" onclick="toggleAllLevels(this)">Voir tous les niveaux</button>`;
    }
  }

  document.getElementById('eval-content').innerHTML = html;
```

- [ ] **Step 7: Verify in browser**

Open `index.html`. Select IC mode, pick "Senior PM" as focus level. Start the assessment. Confirm only 3 level cards are visible (Intermediate, Senior, Staff). Confirm the "Voir tous les niveaux" button appears. Click it — confirm all 5 levels appear and the button text changes to "Masquer les niveaux extrêmes". Click again — confirm levels collapse back. Select a level that is hidden by default (e.g., Junior) and confirm it stays selected if you collapse again (the selected card should always be visible — note: the `level-hidden` CSS is `display:none`, but `selected` cards will be hidden if they're outside the range; this is acceptable since the user won't select an out-of-range level during normal flow). Verify no JS errors.

- [ ] **Step 8: Commit**

```bash
git add index.html
git commit -m "feat: add focus level selector and focused level display (±1)"
```

---

## Task 3: "Points de vigilance" — read-only anti-pattern cards in results

Replace the old misalignment section placeholder with fully rendered read-only cards for all 6 Potential Misalignment dimensions.

**Files:**
- Modify: `index.html` — CSS (vigilance cards), HTML section title, renderResults()

- [ ] **Step 1: Add CSS for vigilance cards**

Find the end of the CSS `</style>` tag. Add before it:

```css
/* Points de vigilance */
.vigilance-intro{font-size:.85rem;color:var(--gray-500);margin-bottom:16px;line-height:1.5;padding:12px 16px;background:var(--gray-50);border:1px solid var(--la-border);border-radius:var(--radius);border-left:3px solid var(--orange-500)}
.vigilance-card{border:1px solid var(--la-border);border-radius:var(--radius-lg);margin-bottom:16px;overflow:hidden;background:#fff;box-shadow:var(--shadow-sm)}
.vigilance-header{padding:14px 16px;background:var(--gray-50);border-bottom:1px solid var(--la-border)}
.vigilance-name{font-weight:600;font-size:.9rem;color:var(--la-dark);margin-bottom:2px}
.vigilance-desc{font-size:.8rem;color:var(--gray-500)}
.vigilance-levels{padding:12px 16px;display:flex;flex-direction:column;gap:10px}
.vigilance-level-name{font-size:.73rem;font-weight:700;text-transform:uppercase;letter-spacing:.04em;color:var(--la-card);margin-bottom:3px}
.vigilance-level-text{font-size:.82rem;color:var(--gray-600);line-height:1.5}
.vigilance-pm{color:var(--gray-400);font-style:italic}
```

- [ ] **Step 2: Update misalignment section header in HTML**

Find in the HTML (results screen):
```html
    <div class="misalignment-section"><h3>Potential Misalignments</h3><div id="misalignments-list"></div></div>
```

Replace with:
```html
    <div class="misalignment-section"><h3>Points de vigilance</h3><div id="misalignments-list"></div></div>
```

- [ ] **Step 3: Replace placeholder with full vigilance card rendering in renderResults()**

Find the placeholder added in Task 1:
```javascript
  // Points de vigilance — placeholder content, full rendering in Task 3
  document.getElementById('misalignments-list').innerHTML = '';
```

Replace with:
```javascript
  // Points de vigilance — all 6 anti-pattern dimensions, read-only
  document.getElementById('misalignments-list').innerHTML = `
    <p class="vigilance-intro">Ces anti-patterns sont présents à tous les niveaux, sous des formes différentes. Aucun n'est à cocher — juste à garder en tête.</p>
    ${MISALIGNMENT_DIMS.map(dim => `
      <div class="vigilance-card">
        <div class="vigilance-header">
          <div class="vigilance-name">${dim.name}</div>
          <div class="vigilance-desc">${dim.description}</div>
        </div>
        <div class="vigilance-levels">
          ${LEVELS.map(level => {
            const data = dim.levels[level];
            return `<div>
              <div class="vigilance-level-name">${level}</div>
              <div class="vigilance-level-text">${data.core}${data.pm ? `<br><span class="vigilance-pm">[PM] ${data.pm}</span>` : ''}</div>
            </div>`;
          }).join('')}
        </div>
      </div>
    `).join('')}
  `;
```

- [ ] **Step 4: Verify in browser**

Complete a full assessment (or use browser console to call `renderResults()` directly after setting `state.mode = 'ic'`). Scroll to the "Points de vigilance" section. Confirm: introductory text is present, all 6 anti-pattern cards are visible, each card shows all 5 levels, there are no checkboxes or select buttons on these cards, and the cards match the expected visual style (header with name + description, then level-by-level content). Verify no JS errors.

- [ ] **Step 5: Commit**

```bash
git add index.html
git commit -m "feat: add Points de vigilance read-only section in results"
```

---

## Notes

- The `selected` state on a `level-hidden` card: if a user somehow has a saved state with a selection outside the ±1 range, the selected card will not be visible by default. This is a non-issue in practice since users set their focus level before evaluating — but the "Voir tous les niveaux" button ensures they can always access it.
- The `focusLevel` field is persisted to localStorage via `saveState()` automatically since it's in the `state` object. Old saved states without `focusLevel` will default to `null` (show all levels), which is a safe fallback.
