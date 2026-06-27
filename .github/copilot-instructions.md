# Copilot Instructions – FantasyDraftLottery

## Project Overview
This is a single-file HTML/CSS/JS Fantasy Draft Lottery app (`index.html`).
All logic, styling, and markup live in that one file.

---

## ✅ Stable Checkpoint — v1.0 (June 2, 2026)

**Everything is working as of this checkpoint.**

- Commit: `63df8add3b19f1d5af4c4098f10aee5f9e383c4f`
- Tag: `v1.0-stable`
- Last meaningful change: Updated title and headers for the 2026 Draft Lottery

### What works
- All draft lottery logic functions correctly
- UI renders and behaves as expected
- No known bugs

### Rollback instructions
If a future change breaks something, you can restore to this checkpoint:
```
git checkout v1.0-stable
```
Or reset `main` to this commit:
```
git reset --hard 63df8add3b19f1d5af4c4098f10aee5f9e383c4f
```

---

## Development Notes for Copilot

- **Single-file architecture**: Keep all changes inside `index.html` unless the user explicitly asks to split into multiple files.
- **No build system**: There is no bundler, package manager, or framework. Keep it plain HTML/CSS/JS.
- **Test before suggesting**: Since there are no automated tests, flag any logic changes that could silently break the lottery draw or team ordering.
- **Preserve working state**: Before making significant changes, note what currently works so regressions are easy to spot.
