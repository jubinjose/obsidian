

# Line Endings Handling

## **Best Fix for Windows + Git**

### Step 1: Configure git to handle the conversion

```bash
git config core.autocrlf true
git config core.safecrlf warn
```

This tells git to:

- Convert LF → CRLF on checkout (so VS Code sees CRLF)
- Convert CRLF → LF on commit (so repo stays clean)
- Warn (not error) on mixed endings

### Step 2: Normalize the repo once

```bash
git add --renormalize .
git commit -m "Normalize line endings to LF in repo"
```

### Step 3: Set Prettier to LF (for consistency)

`.prettierrc.json`:

```json
{
  "endOfLine": "lf"
}
```

Prettier will output LF, but git's `autocrlf` will convert it to CRLF when you edit locally.

### Step 4: Add `.gitattributes` (safety net)

```gitattributes
* text=auto
*.ts text eol=lf
*.js text eol=lf
*.json text eol=lf
```

---

## **Result:**

|Context|Line Ending|
|---|---|
|Your VS Code edits|CRLF (Windows native)|
|Claude's edits|LF (bash)|
|After git commit|LF (git converts CRLF → LF)|
|After git checkout|CRLF (git converts LF → CRLF)|

Git handles the conversion transparently. No mixed endings in the repo, no warnings.
