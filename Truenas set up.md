# TrueNAS SCALE Setup Guide  
> A clean, GitHub-style guide for installing, configuring, and managing TrueNAS SCALE.

TrueNAS is an open-source NAS OS available in two variants:
- **TrueNAS Core** — FreeBSD-based  
- **TrueNAS SCALE** — Linux-based (Debian), recommended for most users due to app support and Kubernetes features

This guide focuses on **TrueNAS SCALE** and includes installation, network configuration, pool creation, and key CLI usage.

---

## 1. Download and Verify ISO

1. Download the latest SCALE ISO:  
   https://www.truenas.com/download-truenas-scale/

2. Verify checksum (Linux/macOS):

   echo "PASTE_SHA256_HERE TrueNAS-SCALE-24.10.iso" | sha256sum --check

## 2. Create Bootable USB (8GB+ recommended)
# Identify USB device
   sudo lsblk        # Linux
   diskutil list     # macOS
# Write ISO to USB
   sudo dd bs=4M if=TrueNAS-SCALE-24.10.iso of=/dev/sdX status=progress oflag=sync
-- ============================================================
--  TrueNAS SCALE INSTALLATION & SETUP GUIDE (SQL FORMAT)
-- ============================================================

 ##  3. Hardware Preparation & Installation
-- ---------------------------------------
#  Minimum Requirements:
--   • 8GB RAM (16GB+ recommended for apps)
--   • Dedicated boot drive (SSD/HDD 32GB+)
--   • One or more data drives for storage pools

#  Installation Steps:
--   1. Boot from USB
--   2. Choose: 1) Install/Upgrade
--   3. Select your OS drive
--   4. Set root password
--   5. Reboot into TrueNAS SCALE

-- ------------------------------------------------------------
 ##  4. Initial Network Configuration (Console Setup)
-- ------------------------------------------------------------
-- After booting, you’ll see the Console Setup Menu.

 #  Option 1 (Recommended) — Set Static IP:
--   Console → 1) Configure Network Interface

 #  Option 2 — Shell (Advanced):
--   Use the following commands (example only):

--   ifconfig eth0 inet 192.168.1.100 netmask 255.255.255.0;
--   route add default 192.168.1.1;
--   echo 'nameserver 8.8.8.8' > /etc/resolv.conf;
--   ping 8.8.8.8;

-- Replace `eth0` with your correct network interface.

-- ------------------------------------------------------------
 ##  5. Access Web UI & Basic Setup
-- ------------------------------------------------------------
-- After networking is configured, access TrueNAS UI via:
--   http://YOUR_TRUENAS_IP

-- First steps:
--   • Update system → System Settings → Update
--   • Create Storage Pool → Storage → Pools → Add
--   • Create Datasets → Storage → Datasets → Add
--   • Create SMB/NFS shares → Sharing
--   • Enable SSH → Services → SSH → Enable

-- ------------------------------------------------------------
 ##  6. Useful Shell Commands
-- ------------------------------------------------------------

-- System Info:
--   midclt call system.version;
--   uname -a;

-- ZFS Storage:
--   zpool status;
--   zpool scrub tank;

-- User Management (Debian-style):
--   adduser newuser;

-- Logs:
--   dmesg | tail;
--   tail -f /var/log/messages;

-- Reboot/Shutdown:
--   reboot;
--   poweroff;

-- Exit shell:
--   exit;

-- ------------------------------------------------------------
 ##  7. Backup Configuration
-- ------------------------------------------------------------
-- Web UI → System Settings → General → Manage Configuration → Download

-- Always back up your configuration after:
--   • Creating pools
--   • Adding users
--   • Configuring shares
--   • Joining AD

-- ------------------------------------------------------------
-- Notes
-- ------------------------------------------------------------
-- • Do NOT use `apt install` on TrueNAS SCALE — unsupported and may break the system.
-- • Use the TrueNAS UI or SCALE API for changes.

