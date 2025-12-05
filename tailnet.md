# 🟦 SOP: How to Join the Lab Using Your Own Tailscale Account  
_For extra team members — no changes needed on the existing Ubuntu VPN server._

---

## ✅ Overview
This guide is for a team member who will:
- Create **their own Tailscale account** (para free)
- Install Tailscale on their own laptop/PC
- Receive the **shared VPN server** from Kristine
- Access Proxmox and all lab servers through that shared access

**IMPORTANT:**  
You will NOT join Kristine’s tailnet.  
She will simply **share the VPN server** with you — unlimited & free.

---

# 1. Create Your Own Tailscale Account

1. Open: https://login.tailscale.com  
2. Click **Sign In**  
3. Choose:
   - Google  
   - Microsoft  
   - or Email login  
4. Once inside, your dashboard will be empty — this is normal.

---

# 2. Install Tailscale on Your Laptop/PC

## **If you use Windows:**
1. Download installer:  
   https://tailscale.com/download/windows
2. Install and open the Tailscale app
3. Sign in using **your OWN Tailscale account** (not Kristine's)

## **If you use Linux (Ubuntu/Debian):**

```bash
curl -fsSL https://tailscale.com/install.sh | sh
sudo tailscale up

