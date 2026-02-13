**Created:** *<span color ="cyan"></span>* <span style="color: green; font-style:italic;"> .., : </span>

**UUID:**
5a8852e0-630a-4e89-ba54-4728e613f3e1

**Task ID:**
5a8852e0-630a-4e89-ba54-4728e613f3e1

**git rev-parse HEAD:**
76f9418e282fcf141fa10c1872bac5fb22477d96

**Github Link:**
https://github.com/gruns/furl

**interface code:**
cc_agentic_coding

---
# Saves and Logs - task xx

## Turn 1

Malformed encoded URLs can slip through silently in one component but warn in another.
### Turn 1 Prompt:

Fix the issue where malformed encoded URLs can silently slip through a component but then suddenly show warning in another component
### Turn 1 Eval Table:

| Question of which is / has           | Answer Given | Justoification Why? |
| ------------------------------------ | ------------ | ------------------- |
| Overall Better Solution              |              |                     |
| Better logic and correctness         |              |                     |
| Better Naming and Clarity            |              |                     |
| Better Organization and Clarity      |              |                     |
| Better Interface Design              |              |                     |
| Better error handling and robustness |              |                     |
| Better comments and documentation    |              |                     |
| Ready for review / merge             |              |                     |

### Pros and cons

#### Model A:
- Pros:
	- `has_valid_percent_encoding()` is nicely modularised and is used in both `Path._segements_from_path()` and `Query._extract_items_from_Querystr()` improving consistency. `warned`in `_segements_from_path()` and `_extract_items_from_Querystr()` gives one single warning per input string. Implementation of `Path`/ `Query` / `Fragment`constructors which means a strict flow is followed. `%`and `%3` etc.; tests check particular edge cases which are generally missed.
	
- Cons:
	- 

#### Model B:
- Pros:
	
- Cons:

#### Justification for best overall

---

## Turn 2

### Turn 2 Prompt:

### Turn 2 Eval Table:

| Question of which is / has           | Answer Given | Justoification Why? |
| ------------------------------------ | ------------ | ------------------- |
| Overall Better Solution              |              |                     |
| Better logic and correctness         |              |                     |
| Better Naming and Clarity            |              |                     |
| Better Organization and Clarity      |              |                     |
| Better Interface Design              |              |                     |
| Better error handling and robustness |              |                     |
| Better comments and documentation    |              |                     |
| Ready for review / merge             |              |                     |

### Pros and cons

#### Model A:
- Pros:
	
- Cons:


#### Model B:
- Pros:
	
- Cons:

#### Justification for best overall

---

## Turn 3

### Turn 3 Prompt:

### Turn 3 Eval Table:

| Question of which is / has           | Answer Given | Justoification Why? |
| ------------------------------------ | ------------ | ------------------- |
| Overall Better Solution              |              |                     |
| Better logic and correctness         |              |                     |
| Better Naming and Clarity            |              |                     |
| Better Organization and Clarity      |              |                     |
| Better Interface Design              |              |                     |
| Better error handling and robustness |              |                     |
| Better comments and documentation    |              |                     |
| Ready for review / merge             |              |                     |

### Pros and cons

#### Model A:
- Pros:
	
- Cons:


#### Model B:
- Pros:
	
- Cons:

#### Justification for best overall
