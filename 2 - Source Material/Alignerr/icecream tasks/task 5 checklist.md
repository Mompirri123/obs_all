# task 5 checklist

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
    
- [x] Confirm `tmux` is installed
    
    `tmux -V`
    
- [x] Install `tmux` if missing
    
- [x] Confirm `claude-hfi` is installed
    
    `claude-hfi --help`
    

---

## 2. CLI Launch

- [x] Decide CLI mode
    
    - [x] VS Code mode (`--vscode`) **(recommended)**
        
    - [ ] All-in-one tmux mode (`--tmux`)
        
- [x]  Start Claude HFI
    
    `claude-hfi --vscode`
    
- [x]  Complete Auth0 authentication
    
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
    
- [x] Watch for:
    
    - [x] Permission prompts
        
    - [x] User input requests
        
    - [x] Errors or blocking states
        
- [x] Exit trajectory shells after completion
    

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
    
    - [ ] Edge cases
        
    - [ ] Naming and clarity
        
    - [ ]  Organization and modularity
        
	- [ ] Error handling
        
- [x] Treat model as junior engineer, not expert
- [x] Complete **at least 2 user turns total**
- [x]  Continue until code is **PR-approvable**
    

---

## 8. Definition of Done Check

- [x] Ready to Submit
	- [ ] Code is production-ready by your judgment
			(or)
	- [x] Or has a ignore-able changes like a README inclusion or very few lines of code to be changed
    
- [x]  No debug prints or commented-out code
    
- [x] Follows codebase style and conventions
    
- [x] Tests pass and coverage is adequate
    
- [x] No further feedback would be required in a real PR
    

---

## 9. Final Diff Generation (After CLI)

- [x] Stage all changes
    
    `git add -A`
    
- [x] Generate final diff using **original commit hash**
    
    `git diff <original-commit-hash> > ~/uuid_final.diff`
    
- [x] Confirm diff applies cleanly at original commit
- [x] Submit final diff
    

---

## 10. Evaluation & Write-Up

- [x] Fill all evaluation axes independently
    
- [x] Provide evidence-based justifications
    
- [x] Write **3–5+ sentences** for overall preference
    
- [x] Avoid mentioning AI, prompts, or models
    
- [x] Submit evaluation