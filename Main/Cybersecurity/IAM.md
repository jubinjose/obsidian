
### Summary Table

|**Category**|**Primary Focus**|**The "Big Players" you listed**|
|---|---|---|
|**IGA**|Compliance & Lifecycle|**SailPoint, Saviynt**|
|**SSO / AM**|Login & Authentication|**Okta, Ping, Microsoft Entra**|
|**PAM**|Protecting Admin Secrets|**CyberArk**|

Here is why each tool belongs in its specific category:

#### 1. The IGA Tools (Governance)

**Tools:** `SailPoint`, `Saviynt`

- **Why here?** These tools are the "auditors." They don't just log you in; they check _if_ you should be allowed to log in. They handle "Joiner, Mover, Leaver" (lifecycle) processes.
    
- **Key Question:** "Does this employee still need access to this data?"
    

#### 2. The Access Management / SSO Tools

**Tools:** `Okta`, `Ping Identity`, `Microsoft Entra`

- **Why here?** These are the "front doors." When a user opens their laptop and needs to get into their email, Slack, or Zoom, these tools check the password (Authentication) and let them in without asking for a password again (SSO).
    
- **Key Question:** "Is this username and password correct?"
    

#### 3. The PAM Tools (Privileged Access)

**Tools:** `CyberArk`

- **Why here?** This is the "vault." It protects the most powerful passwords in the company (like the password to the main server or database). Regular employees don't use this; only IT administrators do.
    
- **Key Question:** "Can we safely let this admin fix the server without showing them the actual password?"
    

---

### Summary Table

| **Category** | **Primary Focus**        | **The "Big Players" you listed** |
| ------------ | ------------------------ | -------------------------------- |
| **IGA**      | Compliance & Lifecycle   | **SailPoint, Saviynt**           |
| **SSO / AM** | Login & Authentication   | **Okta, Ping, Microsoft Entra**  |
| **PAM**      | Protecting Admin Secrets | **CyberArk**                     |