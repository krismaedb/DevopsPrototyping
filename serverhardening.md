Absolutely! Below is a **step-by-step, click-by-click guide** to configure the **Windows Firewall on DC01 and DC02** exactly as required by your NSA630 rubric — including **inbound rules for LDAP, DNS, DHCP, SMB**, **blocking all other traffic**, and **enabling Windows Defender**.

You’ll do this **directly on each Domain Controller** (DC01 and DC02).

---

## ✅ Step 1: Open Windows Defender Firewall with Advanced Security

1. **Log in** to **DC01** as a **Domain Admin** (e.g., `Administrator` or your admin account).
2. Press **`Windows + R`** to open the **Run** dialog.
3. Type:
   ```
   wf.msc
   ```
4. Press **Enter**.
   - This opens **Windows Defender Firewall with Advanced Security**.
   - You should see **Inbound Rules**, **Outbound Rules**, and **Monitoring** in the left pane.

> 🔐 Ensure you’re running this **as Administrator** (you’ll get full control).

---

## ✅ Step 2: Verify Default Policy = **Block All Inbound**

Before adding rules, confirm the default behavior:

1. In `wf.msc`, click **Windows Defender Firewall Properties** in the right-hand pane.
2. A window with **three tabs** appears: **Domain Profile**, **Private Profile**, **Public Profile**.
3. For **Domain Profile** (this is what your DC uses on the clinic network):
   - **Firewall state**: **On**
   - **Inbound connections**: **Block**
4. Repeat for **Private Profile** (set same).
5. **Public Profile** can stay as-is (you won’t use it).
6. Click **OK**.

> ✅ This ensures **only your allowed ports are open** — everything else is blocked by default.

---

## ✅ Step 3: Create Inbound Rules (One by One)

### 🔹 Rule 1: Allow **LDAP (TCP 389)** for Active Directory

1. In `wf.msc`, right-click **Inbound Rules** → **New Rule...**
2. **Rule Type**: Select **Port** → **Next**
3. **Protocol and Ports**:
   - Select **TCP**
   - Choose **Specific local ports**: `389`
   - → **Next**
4. **Action**: **Allow the connection** → **Next**
5. **Profile**: Check **Domain**, **Private** → **Uncheck Public** → **Next**
6. **Name**:  
   ```
   Allow LDAP (AD) - TCP 389
   ```
7. **Description**:  
   ```
   Required for domain authentication and GPO processing
   ```
8. Click **Finish**

---

### 🔹 Rule 2: Allow **DNS (TCP & UDP 53)**

> ⚠️ You must create **two rules**: one for **TCP 53**, one for **UDP 53**.

#### → DNS TCP 53:
- Repeat **Step 3**, but:
  - Protocol: **TCP**
  - Port: `53`
  - Name: `Allow DNS - TCP 53`
  - Description: `DNS queries over TCP`

#### → DNS UDP 53:
- Repeat again:
  - Protocol: **UDP**
  - Port: `53`
  - Name: `Allow DNS - UDP 53`
  - Description: `DNS queries over UDP`

> ✅ Both are needed — Windows DNS uses **both**.

---

### 🔹 Rule 3: Allow **DHCP Server (UDP 67)**

> Only needed on DCs that run **DHCP Server** (which yours do).

1. **New Rule** → **Port** → **Next**
2. Protocol: **UDP**
3. Port: `67`
4. Action: **Allow**
5. Profile: **Domain, Private**
6. Name:  
   ```
   Allow DHCP Server - UDP 67
   ```
7. Description:  
   ```
   Allows clients to request IP addresses from this DHCP server
   ```
8. **Finish**

> 📌 Note: Clients use **UDP 68**, but that’s **outbound** — no inbound rule needed.

---

### 🔹 Rule 4: Allow **SMB/File Shares (TCP 445)**

1. **New Rule** → **Port**
2. Protocol: **TCP**
3. Port: `445`
4. Action: **Allow**
5. Profile: **Domain, Private**
6. Name:  
   ```
   Allow SMB/File Shares - TCP 445
   ```
7. Description:  
   ```
   Enables access to \\healthclinic.local network shares
   ```
8. **Finish**

---

### 🔹 (Optional) Rule 5: Allow **RDP (TCP 3389)** – Only If Needed

> Only add this if you use **Remote Desktop** to manage DCs from another machine.

1. **New Rule** → **Port**
2. Protocol: **TCP**
3. Port: `3389`
4. Action: **Allow**
5. Profile: **Domain, Private**
6. Name: `Allow RDP - TCP 3389`
7. **Finish**

> 🔒 **Security Tip**: After adding, go to **Advanced** tab → **Scope** → restrict to specific IPs (e.g., `10.10.40.100` = your laptop).

---

## ✅ Step 4: Enable Windows Defender Real-Time Protection

1. Open **Start Menu** → type **“Windows Security”** → open it.
2. Click **Virus & threat protection**.
3. Under **“Virus & threat protection settings”**, click **Manage settings**.
4. Ensure these are **ON**:
   - ✅ **Real-time protection**
   - ✅ **Cloud-delivered protection**
   - ✅ **Automatic sample submission**
5. Scroll down → **Exclusions** → **Add or remove exclusions**
   - (Optional) Add `C:\Windows\NTDS` and `C:\Windows\SYSVOL` if you want to exclude AD databases from scans (common in production).
6. Run a **Quick Scan**:
   - Go back → **Scan options** → **Quick scan** → **Scan now**

> 🟢 This fulfills the **“Windows Defender”** requirement in “End Device Hardening”.

---

## ✅ Step 5: Test Your Firewall Rules

### 🔸 Test 1: **Allowed Services Should Work**
From a **client or another VM**:
```powershell
# Test LDAP
telnet 10.10.40.10 389    # Should connect

# Test DNS
nslookup healthclinic.local 10.10.40.10

# Test File Share
\\10.10.40.10\PatientForms   # (after you create it)
```

### 🔸 Test 2: **Blocked Ports Should Fail**
```powershell
# Test HTTP (should fail if not running web server on DC)
telnet 10.10.40.10 80

# Test FTP (should fail)
telnet 10.10.40.10 21
```
> You should see: **“Could not open connection”** or **“Connection refused”**

### 🔸 Test 3: View Firewall Logs (Optional but Professional)
1. In `wf.msc` → **Properties** → **Domain Profile** → **Logging** → **Customize**
2. Set:
   - **Log dropped packets**: **Yes**
   - **Log successful connections**: Yes (optional)
   - **Log file path**: `%systemroot%\system32\logfiles\firewall\pfirewall.log`
3. Click **OK**
4. After a blocked attempt (e.g., `telnet 10.10.40.10 80`), open the log file in Notepad:
   ```
   #Fields: date time action protocol src-ip dst-ip src-port dst-port
   2025-12-11 14:30:12 DROP TCP 10.10.20.100 10.10.40.10 50123 80
   ```

---

## ✅ Step 6: Document for Your GitLab README.md

Take screenshots of:
- `wf.msc` showing all 4–5 inbound rules
- **Windows Defender** showing Real-time protection **ON**
- **Command prompt** showing `telnet 10.10.40.10 80` **failing**
- Successful `nslookup` or file share access

Add to your report:

```markdown
### End Device Hardening – Windows Firewall (DC01 & DC02)

**Rules Configured**:
- Allow TCP 389 (LDAP)
- Allow TCP/UDP 53 (DNS)
- Allow UDP 67 (DHCP Server)
- Allow TCP 445 (SMB/File Shares)

**Default Policy**: Block all other inbound traffic  
**Windows Defender**: Real-time protection enabled

**Testing**:
- ✅ LDAP, DNS, SMB accessible  
- ❌ HTTP/FTP blocked as expected

![Firewall Rules](screenshots/dc01-firewall-rules.png)
![Telnet Blocked](screenshots/telnet-80-blocked.png)
```

---

## 🔄 Repeat for **DC02**

Do **exactly the same steps** on **DC02** (10.10.40.11).  
Both DCs must be hardened.

---

By completing this, you will **secure full 5/5 points** for **“End Device Hardening”** and support your **GPOs, shares, and segmentation** demos.

Let me know when you’ve done this — I’ll guide you next through **creating AD groups and GPOs**! 💻🛡️
