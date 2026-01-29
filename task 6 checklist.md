# task 6 checklist

## 0. Task Preparation

- [ ] Select a **single, atomic task** (bug or enhancement)
- [ ] Verify scope fits a **single PR** 
- [ ] Ensure task is defined from **user perspective**, not implementation details

---

## 1. Git & Environment Prerequisites (Before CLI)

- [ ] Navigate to repository root 
    `cd $(git rev-parse --show-toplevel)`
    
- [ ] Confirm current directory is git root
    `git rev-parse --show-toplevel`
    
- [ ] Record starting commit hash
    `git rev-parse HEAD`
    
- [ ] Save commit hash for final diff generation
- [ ] Confirm `tmux` is installed
    `tmux -V`
    
- [ ] Install `tmux` if missing
- [ ] Confirm `claude-hfi` is installed
    `claude-hfi --help`


---

## 2. CLI Launch

- [ ] Decide CLI mode    
    - [ ] VS Code mode (`--vscode`) **(recommended)**
    - [ ] All-in-one tmux mode (`--tmux`)
    
- [ ]  Start Claude HFI
    `claude-hfi --vscode`
    
- [ ]  Complete Auth0 authentication
- [ ]  Return to control prompt

---

## 3. First Turn (Initial Prompt)

- [ ]  Prompt is **well-scoped and atomic**    
- [ ]   Prompt defines **WHAT** and **WHY**, not **HOW**
- [ ] No explicit instruction to “make code production-ready”
- [ ] No prescriptive implementation details
- [ ] Submit first-turn prompt
- [ ] Wait for Model A and Model B responses

---

## 4. Monitor Trajectories (Required)

- [ ] Note tmux session IDs printed in control terminal
- [ ] Open integrated terminal in **Trajectory A VS Code window**
- [ ] Attach to session A
    `tmux attach -t <session-id>-A`
    
- [ ] Open integrated terminal in **Trajectory B VS Code window**
- [ ] Attach to session B
    `tmux attach -t <session-id>-B`
    
- [ ] Watch for:
    - [ ] Permission prompts
    - [ ] User input requests
    - [ ] Errors or blocking states    
- [ ] Exit trajectory shells after completion

---

## 5. First-Turn Review & Feedback

- [ ] Inspect file diffs in VS Code (both trajectories)
- [ ]  Review control flow, correctness, and structure
- [ ]  Submit required feedback in control terminal 
- [ ] Winner’s changes synced to main repo automatically

---

## 6. Upload Codebase Snapshot (Once Only)

- [ ] Copy codebase after first-turn feedback
    `cp -r . /tmp/<codebase-name>`
    
- [ ] Navigate to copied codebase
    `cd /tmp/<codebase-name>`
    
- [ ] Remove unnecessary directories
    - [ ] node_modules
    - [ ] venv
	- [ ] build artifacts
        
- [ ] Create tar archive    
    `tar -cf <codebase-name>.tar ./<codebase-path>`
    
- [ ] Upload tar file via submission form
    

---

## 7. Successive Turns (PR Reviewer Mode)

- [ ]  Provide **specific, prescriptive feedback**
- [ ] No scope creep introduced
- [ ] Feedback addresses:
    - [ ] Edge cases
    - [ ] Naming and clarity
    - [ ]  Organization and modularity
	- [ ] Error handling
    
- [ ] Treat model as junior engineer, not expert
- [ ] Complete **at least 2 user turns total**
- [ ]  Continue until code is **PR-approvable**
    

---

## 8. Definition of Done Check

- [ ]  Code is production-ready by your judgment
- [ ]  No debug prints or commented-out code    
- [ ] Follows codebase style and conventions
- [ ] Tests pass and coverage is adequate
- [ ] No further feedback would be required in a real PR

---
## 9. Evaluation & Write-Up

- [ ] Fill all evaluation axes independently
- [ ] Provide evidence-based justifications
- [ ] Write **3–5+ sentences** for overall preference
- [ ] Avoid mentioning AI, prompts, or models
- [ ] Submit evaluation

---
## 10. Final Diff Generation (After CLI)

- [ ] Stage all changes
    `git add -A`
    
- [ ] Generate final diff using **original commit hash**
    `git diff <original-commit-hash> > ~/uuid_final.diff`
    
- [ ] Confirm diff applies cleanly at original commit
- [ ] Submit final diff
    

---

