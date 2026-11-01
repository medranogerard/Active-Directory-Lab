# Windows Server & Active Directory Home Lab Documentation

A step-by-step technical log tracking the complete deployment, infrastructure design, and PowerShell provisioning sequence inside a virtualized home lab environment.

## Environment Topology & Grounding
* **Hypervisor:** [Your VM Software, e.g., VMware Workstation / VirtualBox]
* **Operating System:** Windows Server 2022 Datacenter Evaluation Edition
* **Target Domain:** `corp.local`
* **Administrative Context:** Executed via Elevated Administrator PowerShell Session

---

## 🛠️ Step 1: Active Directory Directory Engineering
To prepare the namespace hierarchy for regional staffing, an Organizational Unit (OU) structure must be established beneath the core root forest.

### Create the HR Department OU
```powershell
New-ADOrganizationalUnit -Name "HR Department" -Path "DC=corp,DC=local"
```
*Note: Ensure the domain string exactly targets `DC=corp,DC=local`. Typographical path mismatches will trigger internal `ADException` validation drops.*

---

## Step 2: Encrypted Account & Identity Provisioning
Active Directory enforces domain-wide complexity rules. Creating enabled user accounts requires binding a cryptographic plaintext conversion routine alongside account initialization parameters.

### 1. Initialize Cryptographic String Conversion
```powershell
$SecurePassword = ConvertTo-SecureString "ComplexP@ss123!" -AsPlainText -Force
```

### 2. Generate Account Object Inside the Target OU
```powershell
New-ADUser -Name "John Doe" -GivenName "John" -Surname "Doe" -SamAccountName "jdoe" -UserPrincipalName "jdoe@corp.local" -Path "OU=HR Department,DC=corp,DC=local" -AccountPassword $SecurePassword -Enabled $true
```

### 3. Enforce Interactivity Policies
Forces the newly created identity object to dynamically prompt for password updates upon initial logon sequence:
```powershell
Set-ADUser -Identity "jdoe" -ChangePasswordAtLogon $true
```

---

Step 3: Access Control & Group Architecture
Security group scopes must be deployed natively inside the local administrative boundaries before binding identity objects as security members.

### 1. Instantiating Global Security Group
```powershell
New-ADGroup -Name "HR-Staff" -SamAccountName "HR-Staff" -GroupScope Global -GroupCategory Security -Path "OU=HR Department,DC=corp,DC=local"
```

### 2. Map Account Member Objects
```powershell
Add-ADGroupMember -Identity "HR-Staff" -Members "jdoe"
```

---

Step 4: Verification Logs & Output Testing
Run the following validation sequence to confirm inheritance paths and check directory placement constraints:

```powershell
# Check identity placement trail
Get-ADUser -Identity "jdoe" -Properties DistinguishedName | Select-Object Name, DistinguishedName

# Audit group memberships
Get-ADPrincipalGroupMembership -Identity "jdoe" | Select-Object Name
```

### Expected Output Logs:
```text
Name       DistinguishedName
----       -----------------
John Doe   CN=John Doe,OU=HR Department,DC=corp,DC=local

Name
----
Domain Users
HR-Staff
```
