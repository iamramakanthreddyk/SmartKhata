## Description
<!-- What feature/fix does this PR add? -->

---

## 🔒 DOCUMENTATION ENFORCEMENT CHECKLIST
**REVIEWER: Do NOT approve unless ALL boxes are checked**

### Existing Documentation Rules
- [ ] **No new `.md` files created** (except with team lead approval)
- [ ] **Updated existing docs only:**
  - [ ] `docs/core/02_FEATURES.md` (if new feature)
  - [ ] `docs/core/01_ARCHITECTURE.md` (if schema/layer changed)
  - [ ] `docs/core/03_API.md` (if new endpoints)
  - [ ] `docs/core/ERROR_CODES.md` (if new error codes)
  - [ ] `docs/core/ARCHITECTURE_DECISION_MATRIX.md` (if new pattern)
  - [ ] `docs/core/TROUBLESHOOTING.md` (if new common problem)

### Red Flags (Block PR if ANY are true)
- [ ] ❌ NO files with these patterns:
  - `*_FINAL.md`, `*_COMPLETE.md`, `*_STATUS*.md`
  - `*_v1.md`, `*_v2.md`, `*_UPDATED.md`
  - Dates in filenames: `feature_2026_01.md`
  - Multiple variants: `FEATURE_X_SPEC.md` AND `FEATURE_X_DESIGN.md`

- [ ] ❌ NO new index files (we have 1 README.md)

- [ ] ❌ NO new README.md files in any docs/ subfolder

- [ ] ❌ NO summary of summary files

### Documentation Count Check
- [ ] Total `.md` files still ≤ 20 (run: `find . -name "*.md" | wc -l`)
- [ ] If added doc → Deleted old doc (zero-sum rule)

---

## Code Quality
- [ ] Code follows 6-layer architecture
- [ ] Tests added/updated
- [ ] No duplicate code
- [ ] Error handling implemented

---

## Reviewer Instructions

**Before approving:**

1. **Check documentation** (use checklist above)
   ```bash
   # Verify file count hasn't grown
   find . -name "*.md" -type f | grep -v ".git" | wc -l
   # Should be ≤ 20
   ```

2. **Verify updates** (not new creations)
   ```bash
   # Check git diff for new .md files outside docs/core/
   git diff --name-status --cached | grep "\.md"
   ```

3. **Run sanity check**
   ```bash
   # Make sure docs still reference each other
   grep -r "02_FEATURES" docs/core/README.md
   grep -r "01_ARCHITECTURE" docs/core/README.md
   ```

4. **If documentation is wrong:** 
   - Request changes
   - **Do NOT approve**

---

## Self-Check (Author)

Before submitting:
- [ ] I read `docs/core/HOW_TO_ADD_FEATURES.md`
- [ ] I checked `docs/core/02_FEATURES.md` for duplicates
- [ ] I updated existing docs, not created new ones
- [ ] I didn't create: _FINAL, _COMPLETE, _STATUS, _v2, or date-named files
- [ ] I ran `find . -name "*.md" | wc -l` and verified count ≤ 20

---

## Type of Change
- [ ] Bug fix
- [ ] New feature
- [ ] Breaking change
- [ ] Documentation update

---

## Related Issues
Closes: #

---

**Note:** This template is enforced by Git hooks. PRs that violate these rules will be blocked automatically.
