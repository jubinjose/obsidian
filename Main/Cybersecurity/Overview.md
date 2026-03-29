

# The Total Cybersecurity Framework: Identity vs. Infrastructure

Modern cybersecurity is divided into two primary "buckets" that work together to achieve a **Zero Trust Architecture** (a security model based on continuous verification).

![Zero Trust Architecture diagram, AI generated](https://encrypted-tbn3.gstatic.com/licensed-image?q=tbn:ANd9GcTfsw4RYpSHD-ejQbD4jc9vQa2p5nEcVceo_vOgIjIDpAOn4WAk2MBYfEng8VxStkBJGcSIbcwctEyG3hrX7hQkAnayCW-qRo8G7B8cQJbxr7B-lSY)

Shutterstock

1. **Bucket 1: Identity & Access Management (IAM)** - Focuses on protecting the _User_ (People & Permissions).
    
2. **Bucket 2: Infrastructure Security** - Focuses on protecting the _Environment_ (Systems, Networks, & Devices).
    

---

## Bucket 1: Identity and Access Management (IAM)

IAM answers the question: _"Are you who you say you are, and are you allowed to be here?"_

IAM is split into two distinct audiences: **Workforce IAM** (for internal employees) and **CIAM** (for external customers).

## Division A: CIAM (Customer Identity & Access Management)

- **Focus:** Managing external users, consumers, or citizens accessing a company's web portals or applications. It prioritizes scale, user experience, and privacy.
    
- **Key Question:** "How do we make signing up and logging in as fast, frictionless, and secure as possible for our customers?"
    
- **Role:** The "Digital Storefront."
    
- **Common Tools:** Auth0 (owned by Okta), Microsoft Entra External ID, Ping Identity.
    
- **Key Features:** * Social Login (Sign in with Google, Apple, Facebook).
    
    - Passwordless authentication (Magic links, biometrics).
        
    - Consent and Data Privacy management (GDPR, CCPA compliance).
        

## Division B: Workforce IAM (The 3 Pillars)

Workforce IAM requires strict control over internal user lifecycles and is divided into three main sub-categories:

#### 1. IGA (Identity Governance & Administration)

- **Focus:** Compliance and Lifecycle ("Joiner, Mover, Leaver" processes).
    
- **Key Question:** "Who has access to what, and do they still need it?"
    
- **Role:** The "Auditor." It ensures users only have the permissions necessary for their current job role.
    
- **Common Tools:** SailPoint, Saviynt.
    

#### 2. Access Management (Authentication)

- **Focus:** Login verification and visibility for employees.
    
- **Key Question:** "Are you providing the correct credentials?"
    
- **Role:** The "Front Door."
    
- **Common Tools:** Okta, Ping Identity, Microsoft Entra ID.
    
- **Sub-features and Mechanisms:**
    
    - **SSO (Single Sign-On):** Allows a user to log in once to access all their work apps.
        
    - **SAML 2.0:** An enterprise protocol for passing authorization credentials to service providers.
        
    - **OIDC / OAuth:** Modern protocols. OAuth handles _authorization_ (permissions), while OIDC handles _authentication_ (identity verification).
        

#### 3. PAM (Privileged Access Management)

- **Focus:** Managing and securing super-user/admin accounts.
    
- **Key Question:** "Can we safely let this IT admin fix the server without exposing the master password?"
    
- **Role:** The "Vault." Regular employees do not use PAM. It protects the "Keys to the Kingdom."
    
- **Common Tools:** CyberArk, BeyondTrust.
    

---

## Bucket 2: Infrastructure Security

Infrastructure Security answers the question: _"Is the environment itself safe from attacks, breaches, and data loss?"_

If IAM is the ID badge required to enter a building, Infrastructure Security represents the physical walls, the security cameras, the armored trucks, and the bank vault.

## The 3 Pillars of Infrastructure Security

#### 1. Device Security (Endpoint)

- **Focus:** Protecting the physical machine (laptops, desktops, servers) from malware, ransomware, and unauthorized access.
    
- **Role:** The "Bodyguard." It lives directly on the hardware to stop threats at the source.
    
- **Common Tools:** CrowdStrike, SentinelOne.
    

#### 2. Network Security (Perimeter & SASE)

- **Focus:** Protecting the connection and the traffic flowing between users, the internet, and company servers.
    
- **Role:** The "Traffic Cop" and the "Castle Wall." It stops users from uploading sensitive data to unauthorized places and blocks malicious incoming traffic.
    
- **Common Tools:** Netskope (Cloud/Edge security), Fortinet (Firewalls/Network security).
    

#### 3. Data Security

- **Focus:** Protecting the information itself, regardless of where it lives, and ensuring it can be recovered if something goes wrong.
    
- **Role:** The "Insurance Policy" and the "Vault." It handles data storage, encryption, and backups.
    
- **Common Tools:** Rubrik (Backup & Recovery).
    

---

## Summary Table: Putting it all together

|**Category**|**Sub-Category / Pillar**|**Primary Focus**|**Industry Examples**|
|---|---|---|---|
|**Identity**|CIAM|Customer Logins, Frictionless UX, Privacy|Auth0 (Okta), Ping|
|**Identity**|Workforce: IGA|Governance, Compliance, Lifecycle|SailPoint, Saviynt|
|**Identity**|Workforce: Access Mgmt|Employee Authentication, SSO|Okta, Ping, MS Entra|
|**Identity**|Workforce: PAM|Admin Passwords, Vaulting|CyberArk, BeyondTrust|
|**Infrastructure**|Device (Endpoint)|Laptop & Server Protection|CrowdStrike, SentinelOne|
|**Infrastructure**|Network (SASE)|Traffic, Perimeter, Cloud Edge|Netskope, Fortinet|
|**Infrastructure**|Data|Backups, Resilience, Storage|Rubrik|