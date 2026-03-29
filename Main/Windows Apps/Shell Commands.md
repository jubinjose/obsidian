
## 1. Identify the Process (PID)

Find which Process ID (PID) is currently squatting on a specific port.
```PowerShell
# Lists all connections; 'findstr' filters for the specific port
netstat -ano | findstr :8080
```

## 2. Terminate the Process (Taskkill)

Kill the process once you have the PID from the command above.
```PowerShell
# /F forces the termination
taskkill /PID 2660 /F
```

## 3. The "One-Shot" Shortcut (Node.js)

If you have Node installed, use this to kill a port without looking up the PID.  

```PowerShell
npx kill-port 8080
```

## 4. The Optional - Find process name by pid

```PowerShell
`Get-Process -Id <pid>`
```

## 📂 File System Operations (Command Prompt)

## Force Delete Folder

Use the `RD` (Remove Directory) command in CMD for folders that won't budge.
/S removes subdirectories, /Q is 'Quiet' (no prompt)
```bat
RD /S /Q "D:\node_modules"
```






