# Automated bare-metal installation of DietPi

To automate the bare-metal installation on your Raspberry Pi Model B while ensuring it remains lightweight, the best method is using **DietPi's built-in automation engine**.

Unlike standard Raspberry Pi OS, DietPi allows you to pre-configure software installations (like Node-RED and Mosquitto) in a simple text file before you even plug the SD card into the Pi.

### 1. The Automation Workflow

1. **Flash the Image:** Download the DietPi image for Raspberry Pi (ARMv6/1) and flash it to your SD card using Raspberry Pi Imager or Etcher.
2. **Edit the Config Files:** Before ejecting the SD card from your computer, you will modify two files in the `/boot` partition (which is visible on Windows/Mac/Linux).
3. **First Boot:** Insert the card into the Pi and power it on. DietPi will automatically update itself, install your software, and configure your network without you ever touching a keyboard.

---

### 2. Configure `dietpi.txt`

Open the `dietpi.txt` file on your SD card. This file controls the "auto-install" behavior. Change or verify these specific lines:

```bash
# ----- Network Options -----
AUTO_SETUP_NET_ETHERNET_ENABLED=1
AUTO_SETUP_NET_WIFI_ENABLED=1  # Set to 1 if using Wi-Fi
AUTO_SETUP_NET_WIFI_COUNTRY_CODE=US # Use your 2-letter country code

# ----- Automation Options -----
# 0 = Manual, 1 = Full Automation (no user input)
AUTO_SETUP_AUTOMATED_INSTALL=1

# ----- Software Installation -----
# 122 = Node-RED, 123 = Mosquitto
AUTO_SETUP_INSTALL_SOFTWARE_ID=122,123

# Set a global password for everything (Default is 'dietpi')
AUTO_SETUP_GLOBAL_PASSWORD=your_secure_password

```

> **Note:** If you are using Wi-Fi, also open `dietpi-wifi.txt` and enter your SSID and Password there.

---

### 3. Summary of Automated Tasks

When you power on the Pi with these settings, it performs the following sequence:

* **Expansion:** It automatically expands the filesystem to fill your SD card.
* **Updates:** It runs `apt update` and `apt upgrade` to ensure the OS is current.
* **MQTT (Mosquitto):** It installs Mosquitto and configures it as a background service.
* **Node-RED:** It installs Node.js and Node-RED, then sets up the `systemd` service so it starts on every boot.

---

### 4. Alternative: The "Cloud-Init" Method (Newer Pi OS)

If you prefer the **Official Raspberry Pi OS Lite**, the latest versions (as of 2024/2025) now support `cloud-init`.

1. When using **Raspberry Pi Imager**, click the **Gear Icon** (Advanced Options).
2. Set your username, Wi-Fi, and enable SSH.
3. In the "Run Command" or a custom `user-data` file, you can add:
```yaml
packages:
  - mosquitto
  - mosquitto-clients
runcmd:
  - curl -sL https://raw.githubusercontent.com/node-red/linux-installers/master/deb/update-nodejs-and-nodered | bash -s -- --confirm-install --confirm-pi --notest

```
