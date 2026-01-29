**Created:** *<span class ="color-green">10.12.25, 08:38</span>*

**Note Type:** #atomic  #EnDeNotes

**Hashtags:**
- #cpp
- #controlflow
- #atomic
- #branches

**Links / Tags:**  
- **Relevance Links:**  
    - [[Switch Statements in C++]]      <!-- tightly related sibling -->
      
- **Topic Tags:**  
    - #conditional
    - #booleanlogic

---

# If else in C++

> part of C++ control flow.

`if` and `else` choose **one path** based on a condition that evaluates to `true` or `false`.

## Basic form

### simple if

```cpp
if (condition)
{
	// runs only if condition is true
}
```

### if - else

```cpp
if (condition)
{
	// runs only if condition is true
}
else
{
	// false branch executes when if (if, elseif) blocks fail
}
```

### Chained else-if

```cpp
if (a)
{
    ...
}
else if (b)
{
    ...
}
.
.
.
else
{
    ...
}
```

## key ideas

- the condition is treated as **bool**  
	- any non-zero or non-null expression counts as `true`.
    
- only **one** branch runs
	- once a condition is true, the rest are skipped.
    
- braces are recommended  
	- avoids accidental logic errors when adding statements.

## common usage

- comparisons
- input checks
- guarding code paths
- fallback / default handling

## related concepts

- [[Switch Statements in C++]]
	- multi-way branching when using integral or enum values.


# External References


---
---
# German Version

> Teil der Kontrollstrukturen in C++

- `if` und `else`, helfen dir, einen Weg im program zu wählen
- Die Bedingung entscheidet:
	- `true` = läuft, `false` = läuft nicht.

## einfaches 'if'

```cpp
if (Bedingung)
{
	// läuft nur, wenn Bedingung wahr ist
}
```

## if - else

```cpp
if (Bedingung)
{
	// wird ausgeführt, wenn Bedingung wahr ist
}
else
{
	// passiert, wenn Bedingung falschist
}
```

## if - else if - else

```cpp
if (a)
{
	... 
}
else if (b)
{
	...
}
else
{
	...
}
```

## wichtige punkte

- Die Bedingung ist `bool` (the condition is `bool`)
	- alles Nicht-Null gilt als `true`
- Nur **ein** block läuft
	- Wenn ein Block passt, werden die anderen übersprungen
- Klammen `{}` sind gut, damit keine Fehler passieren

## wofür man es nutzt

- Dinge vergleichen
- Eingaben prüfen
- Fehler vermeiden
- Standardfall festlegen

## wofür man es nutzt

- Dinge vergleichen
- Eingaben prüfen
- Fehler vermeiden
- Standardfall festlegen
