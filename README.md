# NETWORKWALKS-BO83-WK1-PM1-CYBERSECURITY-LAB-SETUP
# Kali Linux Cybersecurity Lab Setup — README

This guide documents the full setup process for the **WK1-PM1 Cybersecurity Lab** task: building a VirtualBox-based ethical hacking lab with Kali Linux, including the issues encountered and how they were fixed.

---

## Task Requirements

- VirtualBox (latest version) as the virtualization base
- Kali Linux configured as the attacking/hacker machine
- Custom **NAT Network** in subnet `10.0.0.0/24`
- Kali Linux static IP: `10.0.0.2/24`
- Full internet access from Kali
- Bidirectional clipboard & drag-and-drop enabled
- Shared folder: host's `Downloads` folder mounted into Kali
- A snapshot taken once setup is complete

---

## Setup Steps

### 1. Install prerequisites
- **7-Zip**: https://www.7-zip.org (used to extract the Kali `.7z` archive)
- **VirtualBox**: https://virtualbox.org/wiki/Downloads

### 2. Extract the Kali VM archive properly
Download the Kali VirtualBox image from https://kali.org/get-kali (`.7z` file).

**Important:** Right-click the `.7z` file and choose **Extract Here** (or "Extract to folder") to a **permanent** location such as `Downloads\kali-linux-2026.2-virtualbox-amd64\`.

Do **not** open the `.vbox` file directly from inside the archive viewer — this silently extracts files into a temporary Windows folder (`AppData\Local\Temp\...`) that gets cleared automatically, which later causes the VM to become **Inaccessible** or **Aborted** with a `VERR_PATH_NOT_FOUND` / `VERR_FILE_NOT_FOUND` error.

After extraction, confirm the folder contains **both**:
- `kali-linux-2026.2-virtualbox-amd64.vbox` (small config file)
- `kali-linux-2026.2-virtualbox-amd64.vdi` (large disk image, several GB)

### 3. Register the VM in VirtualBox
`Machine > Add`, then browse to the extracted folder and select the `.vbox` file.

### 4. Create the NAT Network
`File > Tools > Network Manager > NAT Networks tab > Create`
- Name: `NatNetwork`
- IPv4 Prefix: `10.0.0.0/24`
- Enable DHCP: checked

### 5. Configure the Kali VM's settings
In VM **Settings**:
- **Network > Adapter 1**: Attached to `NAT Network`, Name = `NatNetwork`
- **General > Advanced**: Shared Clipboard = `Bidirectional`, Drag'n'Drop = `Bidirectional`
- **Shared Folders**: Add folder, Path = host's `Downloads` folder, Auto-mount checked, Make Permanent checked

### 6. Log in to Kali
Default credentials: `kali` / `kali`

### 7. Set Kali's static IP

Open a terminal in Kali and run:

```bash
nmcli connection show
sudo nmcli connection modify "Wired connection 1" ipv4.addresses 10.0.0.2/24 ipv4.gateway 10.0.0.1 ipv4.dns 8.8.8.8 ipv4.method manual
sudo nmcli connection down "Wired connection 1"
sudo nmcli connection up "Wired connection 1"
ip a
ping -c 4 8.8.8.8
ping -c 4 google.com
```

### 8. Take a snapshot
Once internet and IP are confirmed working, right-click the VM in VirtualBox Manager, go to the Snapshots tab, and click Take.

- Name: `Fresh Kali - Network Configured`
- Description: notes on NAT Network, static IP, shared folder, and clipboard settings

---

## Common Errors & Fixes

### VM shows "Inaccessible" or "Aborted" — VERR_PATH_NOT_FOUND / VERR_FILE_NOT_FOUND

**Cause:** The `.vbox`/`.vdi` files were opened straight from inside the `.7z` archive, landing in a temporary Windows folder that was later cleared.

**Fix:**
1. Right-click the VM in VirtualBox Manager, choose Remove, then Remove only
2. Fully extract the `.7z` to a permanent folder (e.g. Downloads) using Extract Here
3. Machine > Add, and select the `.vbox` file from that permanent folder

### `nmcli connection up` fails: "IP configuration could not be reserved"

**Cause:** Known issue on VirtualBox v7 with Kali 2026.1+ — a duplicate-address-detection (DAD) check times out.

**Fix:**
```bash
sudo nmcli connection modify "Wired connection 1" ipv4.dad-timeout 0
sudo nmcli connection down "Wired connection 1"
sudo nmcli connection up "Wired connection 1"
```

### Ping fails with "Destination Net Unreachable" from 10.0.0.1

**Cause:** The NAT Network gateway itself can't route out — usually because the host machine's own internet connection (WiFi/Ethernet) was down at the time.

**Fix:** Confirm the host has working internet, then retest:
```bash
ping -c 4 8.8.8.8
ping -c 4 google.com
```

### 7-Zip error: "Cannot open file as archive"

**Cause:** Attempting to extract the `.vbox` file itself — it's a plain XML config file, not an archive. This is expected and harmless; the `.7z` extraction already completed successfully if the `.vbox` and `.vdi` files exist on disk.

---

## Quick Reference: Login & IP Details

| Item | Value |
|---|---|
| Kali username | kali |
| Kali password | kali |
| NAT Network subnet | 10.0.0.0/24 |
| Kali static IP | 10.0.0.2/24 |
| Gateway | 10.0.0.1 |
| DNS | 8.8.8.8 (fallback: 10.0.0.1) |

---

Based on the "Lab Setup (for Cyber Security & Ethical Hacking practice)" task from Networkwalks Academy — www.networkwalks.com



