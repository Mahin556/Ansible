Let's dive deep into **`awk '{$1=$1};1'`** – it's a tiny but powerful line, and understanding it will level up your `awk` skills significantly.

---

## 🎯 What it does (in plain English)
It **removes leading/trailing spaces** and **squeezes multiple spaces/tabs** into a single space, effectively "pretty‑printing" the line.

**Example:**
```
Input:  "    Hello    World   "
Output: "Hello World"
```

---

## 🧩 Breaking it down

An `awk` program consists of **`pattern { action }`** pairs.

The command `'{$1=$1};1'` contains **two statements** separated by a semicolon:

1. **`{$1=$1}`** – an action block (no pattern, so it runs on **every line**).
2. **`1`** – a pattern (no action, so it triggers the **default action: `print`**).

Let's dissect each part:

---

### 1. `{$1=$1}` – The Assignment Trick

- **`$1`** refers to the **first field** of the current line (fields are split by the **Input Field Separator**, `FS`, which defaults to any whitespace).
- **`$1=$1`** assigns the value of `$1` back to itself.
- **Why does this matter?**  
  When you assign a value to **any field** (e.g., `$1`, `$2`, or even `$NF`), `awk` **rebuilds the entire line** (`$0`) by joining all fields together using the **Output Field Separator**, `OFS`.

**Default values:**
- `FS` (Input Field Separator) = any whitespace (space, tab, etc.).
- `OFS` (Output Field Separator) = **single space**.

**The process:**
1. `awk` splits `$0` into fields based on `FS` → leading/trailing whitespace is ignored during splitting; multiple spaces/tabs just act as one delimiter.
2. `$1=$1` triggers a rebuild of `$0` → `awk` joins all fields (from `$1` to `$NF`) with `OFS` (a single space).

**Result:** The line gets "normalized" – no leading spaces, no trailing spaces, and multiple spaces become one.

---

### 2. `;1` – The Always‑True Pattern

- The semicolon `;` separates the first statement (`{$1=$1}`) from the second (`1`).
- **`1`** is a numeric constant. In `awk`, **any non‑zero number evaluates to `true`**.
- Since the pattern `1` is always `true`, and no explicit action is given, `awk` performs the **default action** – which is `{ print $0 }`.

**So `;1` is shorthand for `; { print $0 }`.**

---

## 🚀 Putting it together (the execution order)

For each input line:
1. Execute `{$1=$1}` → rebuilds `$0` with single spaces and no leading/trailing spaces.
2. Evaluate `1` → always `true` → print the (now rebuilt) `$0`.

**Result:** The cleaned‑up version of the original line is printed.

---

## 🔍 Important nuance: `$1=$1` vs. `$0=$0`

You might wonder: *Why not just do `$0=$0`?*

- **`$0=$0`** does **not** trigger the rebuild with `OFS` in most `awk` implementations. It simply reassigns the same string to itself, so spaces are **not** squeezed.
- Only **field assignments** (like `$1=$1`, `$2=$2`, or even `$1=$1` followed by nothing) force a record rebuild using `OFS`.

---

## ✍️ Alternative ways to write the same

All of these produce the same output:

1. **`awk '{$1=$1};1'`** – classic idiom.
2. **`awk '{$1=$1; print $0}'`** – explicit `print`.
3. **`awk '{$1=$1} {print}'`** – pattern-action block.
4. **`awk '{gsub(/^ +| +$/, ""); print}'`** – trims edges only, but doesn't squeeze internal multi‑spaces into one. (Different behavior.)
5. **`awk '{print $0}'`** – prints the line **without** trimming (since it doesn't rebuild the record).

---

## 🧪 Let's test with a practical example

**Input file (spaces.txt):**
```
   foo    bar   baz   
  one     two    three
```

**Run:**
```bash
awk '{$1=$1};1' spaces.txt
```

**Output:**
```
foo bar baz
one two three
```

**Explanation:**
- Leading spaces (`   `) removed.
- Multiple spaces between words (`    `) became one.
- Trailing spaces (`   `) removed.

---

## 🤔 Where did this pattern come from?

It's a well‑known **awk one‑liner** in the Unix community. The `$1=$1` assignment rebuilds `$0`, and `1` prints it. Many sysadmins use it as a quick way to "clean up" whitespace without reaching for `sed` or `tr`.

---

## 📌 In your specific command:
```bash
lscpu | grep Socket | cut -d: -f2 | awk '{$1=$1};1'
```
- `cut -d: -f2` gives `"               1"` (with many spaces).
- The `awk` snippet trims those spaces, outputting just `"1"` (clean, compact).

---

## 💡 Bonus: What if you want to keep only `$1` and skip the rest?

If you wanted **only the first word** (without printing the whole rebuilt line), you'd do:
```bash
awk '{$1=$1; print $1}'
```
But for your case, printing the whole rebuilt line `$0` is exactly what you need.

---

Let me know if you'd like me to go even deeper into `FS`, `OFS`, or other `awk` tricks!