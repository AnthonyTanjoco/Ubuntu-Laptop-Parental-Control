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

## Step 3: Install Native Brave Browser & Block Sandboxed Formats
Modern Linux installs software via sandboxed Snap containers by default. Because sandboxed browsers cannot read global system policy configuration files, we must completely remove the Snap version of Brave and install the native Linux APT release. After doing so, we restrict the Snap store completely.

1. Completely purge the default sandboxed version of Brave (if installed):
   ```bash
   sudo snap remove brave
   ```

2. Install the Official Brave Native APT Repository keys and sources:
   ```bash
   sudo curl -fsSLo /usr/share/keyrings/brave-browser-archive-keyring.gpg https://brave.com
   sudo curl -fsSLo /etc/apt/sources.list.d/brave-browser-release.sources https://brave.com
   ```

3. Update your package manager and install the native build:
   ```bash
   sudo apt update && sudo apt install -y brave-browser
   ```

4. Restrict access to the Snap execution daemon so standard users cannot download alternative sandboxed browsers/VPNs:
   ```bash
   sudo chmod 700 /usr/bin/snap
   ```

5. Restrict access to the Flatpak execution daemon (if installed):
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
5. Enforce File Ownership & Deep Restart:
   1. Ensure the system root profile strictly owns the directory path so users cannot edit it:
   ```
      sudo chown -R root:root /etc/brave/policies/
      sudo chmod -R 755 /etc/brave/policies/
      sudo chmod 644 /etc/brave/policies/managed/tor_policy.json
   ```
   3. Hard kill all persistent background browser instances in memory to force a deep configuration re-read:
      ```
      sudo killall brave
      sudo killall brave-browser
      ```
   
   4. Open Brave on the child's account, navigate to brave://policy, and click "Reload policies". All lines will        show Status: OK.


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
