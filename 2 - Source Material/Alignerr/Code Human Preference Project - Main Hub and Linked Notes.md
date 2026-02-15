# Alignerr Code Human Preference Project


> **Purpose of this page**: This is your **main hub**. Use it to navigate to more detailed notes depending on the *type of task* you’re doing (Turn 1 prompting, follow‑up steering, rationale writing, submission, etc.).

---
# [[Code Human Preference ToDo]]

- `claude-hfi --vscode` for hfi setup
- Pick a Repo to work with
	- (prefer based on prompt readiness)
- Create 1 turn-1 prompt

# Project Main hub

## Big Picture

- You are participating in a **Code Human Preference (HFI)** evaluation project
- Your role is to **evaluate AI-generated code changes**, not just write code.
- You will
	* Prompt two AI models
	* Compare their outputs (Model A vs Model B)
	* Explain *why* one is better
	* Submit structured feedback and artifacts
	  
---

## How to Use These Notes

- Use this page as your **entry point**
- Jump to the section that matches what you are doing *right now*:

- ***SECTIONS***
	* 🔧 **Environment / Setup** → Sections 1–5
	* 🧠 **Turn 1 Prompting** → Section 6 + linked Prompting Notes
	* 🔍 **Model Evaluation & Rationale** → Sections 7–9 + Rationale Notes
	* 📦 **Uploads & Submission** → Sections 10–12 + Submission Notes

If something feels unclear, cross‑check with:

* **Project Instructions (official)** ← your authoritative source


-----

## Official Project Instructions (Authoritative)

🔗 [[Official Project Instructions – CLI Edition]]

Use this link when:
- You need the exact wording of project rules
- You are unsure about scope, ratings, or submission steps
- You want to verify whether something is allowed
- You are checking rubric definitions or edge cases

If anything in these notes conflicts with that document,  
**the official instructions always win**.

## Step-by-Step Instructions

### 1. Access Your Mailbox

**When to use this:** First login / session start

* Click **Access Mailbox** in the project UI
* Copy the **special Alignerr email address** shown at the top
* Use this email to log into the feedback portal

---

### 2. Read the Project Instructions (Required)

**When to use this:** Before *every* task, and whenever unsure

Focus on:

* CLI installation
* Prompting constraints
* Rationale & rating rules
* Submission requirements

📌 *Rule of thumb:* If your intuition and the instructions conflict, **the instructions win**.

---

### 3. Install the Cloud HFI CLI

**When to use this:** First-time setup or version updates

1. Go to the **Feedback Portal**
2. Log in using your special Alignerr email
3. Retrieve the login code from your mailbox
4. Download the correct binary:

   * Mac → Darwin
   * Linux → Linux x64
   * Windows → WSL required
5. Move it to a global bin directory:

   ```bash
   ~/.local/bin
   ```
6. Make it executable:

   ```bash
   chmod +x cloud-hfi
   ```

Check Discord to confirm you are on the **required CLI version**.

---

### 4. Choose a Codebase (Repo)

**When to use this:** Before starting a new task

Your repo must:

* Be a git repository
* Match your assigned language
* Be open source
* Have working build/run instructions
* Be reasonably high quality (but imperfect)

Before committing to a repo:

* Browse the code
* Run examples or tests
* Identify *real*, user-visible problems

📌 *Mental check:* “Would a professional engineer realistically work in this repo?”

---

### 5. Clone and Open the Repo

```bash
git clone <repo-url>
cd <repo>
cloud-hfi --vscode
```

This opens:

* Your browser
* Two VS Code windows

Each VS Code window is a **worktree**:

* **Worktree A** → Model A
* **Worktree B** → Model B

---

## Turn-Based Workflow

### 6. Turn 1 – Write Your First Prompt (Most Important)

**When to use this:** Starting a new task

Your Turn 1 prompt should:

* Be written from a **user perspective**
* Describe what is broken, confusing, or missing
* Reference concrete observations
* Avoid prescribing implementation details

**Good example:**

> The examples do not run correctly, rely on magic numbers, and do not explain how users should interact with the system.

**Bad example:**

> Clean up the code and improve naming.

📌 *Goal of Turn 1:* Define the problem space clearly enough that **multiple turns are required**.

---

### 7. Evaluate Model A vs Model B

**When to use this:** After each model finishes

For each model:

* Inspect code changes carefully
* Run examples/tests if possible
* Verify that original issues are *actually fixed*

Watch out for:

* Superficial or band-aid fixes
* Broken abstractions
* Fake features or hallucinated roadmaps

---

### 8. Write Your Rationale

**When to use this:** Before giving ratings

You must write:

* Pros of Model A
* Cons of Model A
* Pros of Model B
* Cons of Model B
* Overall preference justification

Rules:

* Each bullet must be **atomic**
* Do not mix models in a single point
* Base claims on observed behavior
* Ensure rationale aligns with ratings

📌 *Tip:* If unsure, emphasize **negatives**.

---

### 9. Give Ratings

**When to use this:** After rationale is complete

Rate categories such as:

* Overall quality
* Modularity & organization
* Documentation
* Interface design
* Readiness to merge

Tips:

* Use ❓ tooltips for axis definitions
* Ratings should follow your rationale
* Choose A or B honestly — this determines the next turn

Submit feedback for **Turn 1**, then proceed.

---

## Artifacts & Submission

### 10. Upload the Preferred Codebase (After Turn 1)

1. Copy the repo to `/tmp`
2. Remove unnecessary folders (venv, node_modules, etc.)
3. Create a tarball:

   ```bash
   tar cf traffic-simulator.tar ./traffic-simulator
   ```
4. Upload the tarball to the **Google submission form**
5. Provide:

   * Discord username
   * UUID
   * Initial git commit hash

📌 Upload the codebase **only once per task**.

---

### 11. Turn 2 (and Possibly Turn 3)

**When to use this:** Refinement phase

Your follow-up prompts should:

* Stay within the original scope
* Be narrower and more prescriptive
* Sound like real PR review comments

Repeat the same evaluation loop:

* Inspect
* Rationale
* Rate
* Submit feedback

Most submissions are **2 turns**, sometimes 3.

---

### 12. Final Submission

At the end:

* Generate a diff from the original commit
* Apply it to a clean clone to confirm correctness
* Upload:

  * Final diff file
  * Time spent
  * Number of turns
  * Final ratings

Submit the form.

---

## One-Sentence Summary

You prompt AI models to improve a real codebase, rigorously compare their changes, explain your reasoning, and submit structured feedback — **not just code**.
