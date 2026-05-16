# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repo overview

This repository has two independent areas:

1. **Personal resume webpage** — `index.html` + `photo.jpg`, a single static page styled in warm earth tones with dark top bar, two-column card grid, expandable detail sections, scroll-reveal animations, and responsive layout. The user's name is 杨昊翔. Deployed to GitHub Pages at `yhx442.github.io/personal-resume/`.

2. **Robotics simulation labs** — `lab1_puma560.py` (Puma 560 workspace, inverse kinematics, trajectories) and `lab2_polynomial.py` (polynomial joint trajectory planning). Helper demos in `demo1.py`–`demo4.py`.

## Commands

```powershell
# Run lab simulations
python lab1_puma560.py            # Experiment I: kinematics & trajectories
python lab1_puma560.py --animate  #   with animation
python lab2_polynomial.py         # Experiment II: polynomial trajectories

# Install Python dependencies (venv already exists)
.venv/Scripts/pip install -r requirements.txt
```

## Git & deployment

- Remote: `https://github.com/yhx442/personal-resume.git` (main branch)
- Deployed via GitHub Pages. After pushing to main, the resume is live at `https://yhx442.github.io/personal-resume/`
- SSL verification may need to be disabled on this machine: `git -c http.sslVerify=false push`

## Notes for resume edits

- The resume is a single static HTML file (`index.html`). All CSS is in a `<style>` block, all JS in a `<script>` block at the bottom. No frameworks or build steps.
- The photo reference is `<img src="photo.jpg">` — always keep `photo.jpg` alongside `index.html`.
- Card expand/collapse uses an accordion pattern: clicking a card collapses any previously open card, then expands the clicked one.
- Print styles are embedded in a `@media print` block — cards auto-expand on print.
- The user may reference a PDF at `D:\OneDrive\Desktop\0210杨昊翔\杨昊翔简历.pdf` for content verification, but the source of truth is what's visible in the browser.
