

| ENDPOINT SECURITY                                    | NETWORK / SASE                                 | DATA SECURITY                |
| ---------------------------------------------------- | ---------------------------------------------- | ---------------------------- |
| _Focus: The Device (Laptop)_                         | _Focus: The Traffic_                           | _Focus: The Files_           |
| **"Is your laptop infected?"**                       | **"Is the connection safe?"**                  | **"Is the data backed up?"** |
| • **CrowdStrike**<br><br>  <br><br>• **SentinelOne** | • **Netskope**<br><br>  <br><br>• **Fortinet** | • **Rubrik**                 |


#### **1. CrowdStrike & SentinelOne (Endpoint Security)**

- **What they do:** They live on your laptop (the "Endpoint"). They look for viruses, malware, and hackers trying to break into the computer itself.
    
- **Relationship to IAM:** They talk to tools like Okta.
    
    - _Example:_ **CrowdStrike** tells **Okta**: "This laptop has a virus." **Okta** then says: "Okay, I will block this user from logging in until the virus is gone."
        

#### **2. Netskope (SASE / Cloud Security)**

- **What they do:** They sit between the user and the internet (the "Edge"). They stop employees from uploading secret company files to personal Gmail or Dropbox.
    
- **Relationship to IAM:** They enforce the rules after login.
    
    - _Example:_ **Okta** logs you in, but **Netskope** watches what you do and blocks you if you try to steal data.
        

#### **3. Fortinet (Network Security)**

- **What they do:** Famous for Firewalls. They build the digital walls around the company office. (Note: Fortinet does sell some IAM tools, but they are primarily known for Network Security).
    
- **Relationship to IAM:** They protect the pipes the data travels through.
    

#### **4. Rubrik (Data Security)**

- **What they do:** Backups and Recovery. If a hacker gets past everyone else and deletes all your data, Rubrik is the "Undo" button.
    
- **Relationship to IAM:** The last line of defense.