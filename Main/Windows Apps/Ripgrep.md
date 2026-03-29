#terminal #tools #search #productivity

> [!abstract] Summary
> 
> `ripgrep` is a line-oriented search tool that recursively searches your current directory for a regex pattern. It is famously fast and respects your `.gitignore` by default.

---
## 🚀 Quick Reference Table

| **Goal**    | **Command**       | **Description**                              |
| ----------- | ----------------- | -------------------------------------------- |
| **Basic**   | `rg "pattern"`    | Recursive search for regex in current dir.   |
| **Literal** | `rg -F "string"`  | Treats pattern as a fixed string (no regex). |
| **Case**    | `rg -i "pattern"` | Case-insensitive search.                     |
| **Invert**  | `rg -v "pattern"` | Show lines that **do not** match.            |
| **Files**   | `rg -l "pattern"` | List filenames only (no content).            |
| **Hidden**  | `rg -u "pattern"` | Include hidden and `.gitignore` files.       |

---
## 📂 Filtering by File Type

Ripgrep has built-in aliases for common file extensions.

- **Include specific type:** `rg -tpy "pattern"` (Searches only `.py` files)
    
- **Exclude specific type:** `rg -Tjs "pattern"` (Skips all `.js` files)
    
- **Glob pattern:** `rg "pattern" -g "README.*"` (Searches only files matching the glob)
    

---
## 🖼️ Context & Display

Useful when you need to see the code _around_ the match.

- **Context:** `rg -C 3 "pattern"` (Shows 3 lines before and after)
    
- **Before/After:** Use `-B` for before and `-A` for after specifically.
    
- **Unrestricted:** `-uu` will search hidden files, and `-uuu` will even search binary files.
    
---
## 💡 Common Use Cases

## Find TODOs in Code

> [!example] Search for "TODO" in JavaScript files, listing only the filenames:
> 
> Bash
> 
> ```
> rg -tjs -l "TODO"
> ```

## Find a string in a specific file type while ignoring case

Bash

```
rg -i -tgo "func Main"
```

---

> [!tip] Integration
> 
> If you use **fzf**, you can pipe `rg` into it for a blazing-fast interactive file searcher.