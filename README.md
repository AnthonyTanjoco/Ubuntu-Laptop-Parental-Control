# Ubuntu Parental Control & Brave Lockdown Blueprint

**Project Goal:** Secure an Ubuntu laptop for schoolwork and Steam gaming. The parent remains the system administrator. The son uses a Standard User account restricted by CleanBrowsing Family Filter DNS and managed browser policies to block explicit content, alternative browsers, proxies, and VPNs.

---

## Step 1: System-Wide DNS Configuration (CleanBrowsing)
This forces the entire operating system to use the CleanBrowsing Family Filter at the network layer.

1. Open a terminal and edit the global resolver configuration file:
   ```bash
   sudo nano /etc/systemd/resolved.conf
   ```
2. Uncomment the `DNS=` and `FallbackDNS=` lines, then add the **CleanBrowsing Family Filter** IPs:
   ```text
   [Resolve]
   DNS=185.228.168.168 185.228.169.168
   FallbackDNS=2a0d:2a00:1:: 2a0d:2a00:2::
   ```
3. Save the file (`Ctrl+O`, then `Enter`) and exit (`Ctrl+X`).
4. Restart the systemd-resolved engine to apply the rules:
   ```bash
   sudo systemctl restart systemd-resolved
   ```
5. Verify the active DNS configuration:
   ```bash
   resolvectl status
   ```

---

## Step 2: Enforce Standard User Account
Ensure your son does not have administrative (`sudo`) access to tamper with system configuration files.

1. If his account is already created, strip his `sudo` access by running:
   ```bash
   sudo deluser <username> sudo
   ```
   *(Replace `<username>` with his actual Ubuntu account name).*

---

## Step 3: Block Alternative Browser & VPN Installations
Modern Linux allows standard users to install standalone web browsers and VPN tools via Snap or Flatpak backends into their home directory without needing an admin password. Block execution permissions for these tools.

1. Restrict access to the **Snap** execution daemon:
   ```bash
   sudo chmod 700 /usr/bin/snap
   ```
2. Restrict access to the **Flatpak** execution daemon (if installed):
   ```bash
   sudo chmod 700 /usr/bin/flatpak
   ```

---

## Step 4: Secure Brave Browser via Managed Policies
This step locks Brave Browser's network settings, disables hidden proxy loopholes (Tor, IPFS, WebTorrent), locks strict SafeSearch, and blocks malicious browser extensions.

1. Create the system folder structure for Brave policies:
   ```bash
   sudo mkdir -p /etc/brave/policies/managed
   ```
2. Create and edit a new deployment policy file:
   ```bash
   sudo nano /etc/brave/policies/managed/tor_policy.json
   ```
3. Paste the following comprehensive JSON configuration block:
   ```json
   {
     "TorDisabled": true,
     "BraveVPNDisabled": true,
     "BraveWalletDisabled": true,
     "ForceGoogleSafeSearch": true,
     "ForceYouTubeRestrict": 2,
     "ExtensionSettings": {
       "*": {
         "installation_mode": "blocked"
       },
       "ghmbeldphafepmbegfdlkpapadhbakde": {
         "installation_mode": "normal_installed",
         "update_url": "https://google.com"
       }
     }
   }

   ```
4. Save the file (`Ctrl+O`, then `Enter`) and exit (`Ctrl+X`).
5. Open Brave and navigate to `brave://policy` to confirm all entries display a green **Status: OK**.

---

## Step 5: How to Whitelist Specific Browser Extensions
Since all extensions are blocked by default (`*`: `blocked`), use this process when he needs a specific tool (e.g., Google Translate) approved for school.

1. Find the extension on the [Chrome Web Store](https://google.com).
2. Extract the unique **32-character ID** from the very end of the URL.
   * *Example:* For Google Translate, the ID is `aapbdbdomjkkjkaonfhkkikfgjicclbb`.
3. Open the managed policy file:
   ```bash
   sudo nano /etc/brave/policies/managed/tor_policy.json
   ```
4. Modify the `ExtensionSettings` block to include the exception:
   ```json
   "ExtensionSettings": {
     "*": {
       "installation_mode": "blocked"
     },
     "aapbdbdomjkkjkaonfhkkikfgjicclbb": {
       "installation_mode": "normal_installed"
      "update_url": "https://google.com"
     }
   }
   ```
5. Save the file and restart Brave to apply the exception.
