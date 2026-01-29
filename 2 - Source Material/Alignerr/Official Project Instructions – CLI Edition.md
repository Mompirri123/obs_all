# Official Project Instructions – CLI Edition

> **Role of this document**: This is the **authoritative reference** for the Code Human Preferences (CLI Edition) project. When in doubt, **this document overrides personal notes, summaries, or habits**.

---
## Quick Links & Supporting References

These links are **supporting materials**
Use them **when relevant**.

- **Claude Code CLI Binary Download**  
  → Use during first-time setup or version updates

- **CLI Install Instructions**  
  → Reference if the CLI fails to launch or PATH issues occur

- **Submission Form**  
  → Used only at submission time

- **Additional Tips**  
  → Optional guidance; non-authoritative

- **Rationale Writing Guide**  
  → Consult if you struggle to articulate tradeoffs or preferences
---

## 1. Before You Begin

Read these instructions **in full** and return to them often. Do not gloss over details. Many rules are strict, and violations can invalidate a submission.

---

## 2. Prerequisites Required

You must:

* Be proficient with **Git**
* Understand what **production‑ready code** looks like
* Be competent in your assigned programming language
* Have the **Claude Code CLI** installed and working locally

🔗 **[CLI Install Instructions](https://docs.google.com/document/d/1CQzW4R542zI9bJsato8aAWNM4kKKsOV-C4S_CZcw_lc/edit?usp=sharing)**
📝 **[[CLI Install Instructions]]**

---

## 3. Overall Aim

For each task you will:

1. Select a **real, professional git codebase**
2. Ask the model to perform **one well‑scoped task**
3. Iterate across **2+ turns**
4. Steer the model like a **real engineer / PR reviewer**
5. End with code you would genuinely approve for production

You are expected to ensure the model:

* Reviews its own code
* Validates against requirements
* Uses git appropriately
* Produces production‑ready changes

---

## 4. Quick Setup Check

Ensure the following:

* **Claude Code CLI version:** v2.0.70
* **Interface code:** `cc_agentic_coding`

---

## 5. Important Updates & Warnings

### Proper Usage of N/A Rating (Dec 14, 2025)

* Use **middle ratings** when Model A and B are equivalent
* Use **N/A only if the axis truly does not apply**

### Common Things to Avoid (Dec 7, 2025)

* ❗ Do NOT include the `claude-hfi` binary in the codebase
* ❗ Do NOT include your name or PII in uploads
* ❗ Follow‑up turns must **not introduce scope creep**

---

## 6. Selecting a Codebase

### Required Criteria

The codebase must:

* Be a **git repository**
* Be primarily written in your assigned language
* Be **high quality** (not hobby / vibe‑coded)
* Have **multiple contributors or recognition**
* Be **open source licensed**
* Have **clear, working build/run instructions** (you must verify locally)

### Additional Guidance

* Prefer repos with existing tests
* Mix libraries, apps, SDKs
* You may reuse a repo up to **10 times** (vary prompts)
* Read `SKILL.md`, `CLAUDE.md`, `CONTRIBUTING.md` if present
* Avoid repos requiring heavy networking or internet access

---

## 7. Preparing the Codebase

* If setup instructions are missing, **add and commit them**
* Test instructions locally
* Ensure the git worktree is **clean** before starting Claude Code

---

## 8. Familiarize Yourself With the Codebase

You must:

* Understand purpose, structure, and behavior
* Be comfortable reviewing all changes
* Be capable of proposing realistic improvements

---

## 9. Starting Claude Code

Launch using VS Code or tmux and enter:

```
cc_agentic_coding
```

---

## 10. First Turn Prompting Rules (Authoritative)

Your **Turn 1 prompt** must:

* Be sized for a **single PR**
* Be written from a **user perspective**
* Avoid prescribing implementation details
* Be challenging enough to require **multiple turns**
* NOT explicitly say “make this production ready”

### Examples

**Good:**

* Add resume logic so training can continue from saved checkpoints
* Update naming to reflect new filesystem understanding

**Bad:**

* Add dashboards, streaming UI, pause/resume controls
* Specify exact APIs, defaults, or architecture
* Explicitly instruct the model to be production ready

---

## 11. Successive Turns (Turn 2+)

Follow‑up prompts should:

* Maintain the **original scope**
* Be **prescriptive and technical**
* Sound like real **PR review comments**
* Reflect **expert judgment**

Avoid generic steering. Consistently doing so can result in removal from the project.

---

## 12. Definition of Production Ready

Production‑ready code:

* Fully implements requested scope
* Handles edge cases
* Considers security implications
* Matches project style
* Uses appropriate abstractions
* Avoids unnecessary comments or AI chain‑of‑thought
* Includes tests when appropriate

---

## 13. Reviewing Model Outputs

Ensure the model:

* Follows engineering best practices
* Writes appropriate documentation
* Uses git effectively
* Reviews and revises its own work
* Produces merge‑ready changes

Provide feedback as you would on a real PR.

---

## 14. Definition of Done

The interaction is complete when:

* You would approve the PR as a maintainer

---

## 15. Ratings & Annotations

For each turn:

1. Write **pros and cons** for Model A and Model B
2. Focus on negatives when unsure
3. Rate all grading axes
4. Write a **3–5+ sentence overall preference justification**

Overall preference should reflect **all axes combined**.

---

## 16. Submission Steps

### Before Starting

* Run `git rev-parse HEAD` and record the commit hash
* Launch Claude Code and record the UUID

### After Turn 1

1. Copy repo:

   ```bash
   cp -r . /tmp/<codebase-name>
   ```
2. Remove large/unnecessary folders
3. From parent directory:

   ```bash
   tar -cf <codebase-name>.tar ./<codebase-name>
   ```
4. Upload tar to the submission form

### After Final Turn

1. Ensure all files are staged:

   ```bash
   git add -A
   ```
2. Generate diff:

   ```bash
   git diff <commit_hash> > ~/<uuid>_final.diff
   ```
3. Upload final diff

---

## 17. Rubric Axes (Full Reference)

### Logic & Correctness

* Correct behavior
* Edge cases handled
* No subtle bugs

### Naming & Clarity

* Clear, consistent naming
* No ambiguity
* Units and intent obvious

### Organization & Modularity

* Single responsibility
* No duplication
* Reasonable complexity

### Interface Design

* Intuitive APIs
* Predictable behavior
* Minimal coupling

### Error Handling & Robustness

* Proper exceptions
* No swallowed errors
* Defensive where appropriate

### Comments & Documentation

* Explain **why**, not **what**
* No AI chain‑of‑thought
* Updated and useful

### Ready for Review / Merge

* No debug artifacts
* Clean git hygiene
* Tests passing
* Configurable values

---

## Final Reminder

You are evaluated on **judgment, rigor, and realism**, not volume of changes.

When unsure, err toward what a **senior engineer maintainer** would approve.
