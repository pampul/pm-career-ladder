# PM Career Ladder — UX Improvements Design

**Date:** 2026-04-09  
**Status:** Approved

---

## Context

The current self-assessment tool covers 20 dimensions across 5 categories (including 6 "Potential Misalignments"). Two pain points were identified:

1. **Too long** — presenting all 5 levels for every dimension creates visual noise and cognitive fatigue.
2. **Potential Misalignments are awkward to check** — anti-patterns don't map well to a "select your level" interaction; they're better consumed as awareness content.

---

## Changes

### 1. Level focus based on declared level

**Where:** Setup screen (before the assessment starts), alongside existing fields (name, role, etc.).

**What:** Add a new required field — "Quel est ton niveau actuel ou cible ?" — with a select: Junior / Intermediate / Senior / Staff / Principal.

**Behavior:** This declared level drives which 3 levels are shown by default on each dimension card:

| Declared level | Levels shown by default |
|---|---|
| Junior | Junior, Intermediate |
| Intermediate | Junior, Intermediate, Senior |
| Senior | Intermediate, Senior, Staff |
| Staff | Senior, Staff, Principal |
| Principal | Staff, Principal |

> Edge cases (Junior, Principal) show only 2 levels since there is no level before/after.

**Collapsed levels:** The levels outside the default 3 are hidden but accessible. A subtle link — "Voir tous les niveaux" — appears at the bottom of each dimension and expands the full list inline. No navigation away from the current step.

---

### 2. Potential Misalignments moved out of the assessment flow

**Where:** Currently: 6 steps in the stepper (steps 16–21). After: removed from the stepper entirely.

**What:** The 6 Potential Misalignment dimensions (Ownership, Initiative, Scope Awareness, Feedback & Communication, Quality & Detail, Mentorship & Influence) are no longer part of the evaluation flow. They have no level selector, no score contribution, and no skip link.

**New placement:** A dedicated section at the end of the results screen, after strengths/growth areas, titled **"Points de vigilance"**. Each of the 6 dimensions is displayed as a read-only card showing all level content without distinction. No interaction required — awareness only.

The section intro copy should frame it as: *"Ces anti-patterns sont présents à tous les niveaux, sous des formes différentes. Aucun n'est à cocher — juste à garder en tête."*

---

## What does NOT change

- Step-by-step navigation flow (one dimension per screen)
- Scoring logic — only the 15 remaining dimensions contribute to the score
- Results screen structure (score badge, profile map, strengths, growth areas)
- Notes per dimension
- Skip functionality
- Local storage / auto-save
- Mobile layout

---

## Impact summary

| Before | After |
|---|---|
| 5 steps in stepper (1 per category) | 4 steps |
| All 5 levels expanded or collapsed manually | 3 relevant levels shown by default |
| Anti-patterns as checkable steps | Anti-patterns as read-only awareness section |
