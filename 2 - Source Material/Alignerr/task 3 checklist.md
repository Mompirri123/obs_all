# task 3 checklist

## 0. Task Preparation

- [x] Select a **single, atomic task** (bug or enhancement)
- [x] Verify scope fits a **single PR** 
- [x] Ensure task is defined from **user perspective**, not implementation details

---

## 1. Git & Environment Prerequisites (Before CLI)

- [x] Navigate to repository root
    
    `cd $(git rev-parse --show-toplevel)`
    
- [x] Confirm current directory is git root
    
    `git rev-parse --show-toplevel`
    
- [x] Record starting commit hash
    
    `git rev-parse HEAD`
    
- [x] Save commit hash for final diff generation
    

---

## 2. CLI Launch

- [x] Decide CLI mode
    
    - [x] VS Code mode (`--vscode`) **(recommended)**
          
- [x]  Start Claude HFI
    `claude-hfi --vscode`
    
- [x]  Return to control prompt
    

---

## 3. First Turn (Initial Prompt)

- [x]  Prompt is **well-scoped and atomic**
    
- [x]   Prompt defines **WHAT** and **WHY**, not **HOW**
    
- [x] No explicit instruction to “make code production-ready”
    
- [x] No prescriptive implementation details
    
- [x] Submit first-turn prompt
    
- [x] Wait for Model A and Model B responses
    

---

## 4. Monitor Trajectories (Required)

- [x] Note tmux session IDs printed in control terminal
    
- [x] Open integrated terminal in **Trajectory A VS Code window**
    
- [x] Attach to session A
    
    `tmux attach -t <session-id>-A`
    
- [x] Open integrated terminal in **Trajectory B VS Code window**
    
- [x] Attach to session B
    
    `tmux attach -t <session-id>-B`
    
- [ ] Watch for:
    
    - [ ] Permission prompts
        
    - [ ] User input requests
        
    - [ ] Errors or blocking states
        
- [ ] Exit trajectory shells after completion
    

---

## 5. First-Turn Review & Feedback

- [x] Inspect file diffs in VS Code (both trajectories)
    
- [x]  Review control flow, correctness, and structure
    
- [x]  Submit required feedback in control terminal
    
- [x] Winner’s changes synced to main repo automatically
    

---

## 6. Upload Codebase Snapshot (Once Only)

- [x] Copy codebase after first-turn feedback
    
    `cp -r . /tmp/<codebase-name>`
    
- [x] Navigate to copied codebase
    
    `cd /tmp/<codebase-name>`
    
- [x] Remove unnecessary directories
    
    - [x] node_modules
        
    - [x] venv
        
	 - [x] build artifacts
        
- [x] Create tar archive
    
    `tar -cf <codebase-name>.tar ./<codebase-path>`
    
- [x] Upload tar file via submission form
    

---

## 7. Successive Turns (PR Reviewer Mode)

- [x]  Provide **specific, prescriptive feedback**
    
- [x] No scope creep introduced
    
- [x] Feedback addresses:
    - [x] Edge cases
    - [x] Naming and clarity
    - [x]  Organization and modularity
	- [x] Error handling
	      
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

## 9. Final Diff Generation (After CLI)

- [ ] Stage all changes
    
    `git add -A`
    
- [ ] Generate final diff using **original commit hash**
    
    `git diff <original-commit-hash> > ~/uuid_final.diff`
    
- [ ] Confirm diff applies cleanly at original commit
- [ ] Submit final diff
    

---

## 10. Evaluation & Write-Up

- [ ] Fill all evaluation axes independently
    
- [ ] Provide evidence-based justifications
    
- [ ] Write **3–5+ sentences** for overall preference
    
- [ ] Avoid mentioning AI, prompts, or models
    
- [ ] Submit evaluation