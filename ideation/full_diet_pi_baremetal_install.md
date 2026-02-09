# Fully automated bare-metal installation of DietPi

To automate a bare-metal installation on your Raspberry Pi Model B while ensuring your irrigation flows are ready to go the moment it boots, we will use the **DietPi Automation Engine** and a **Post-Install Script**.

This approach allows you to flash an SD card, edit two text files, and walk away while the Pi builds itself.

### 1. Pre-configure the `dietpi.txt`

Flash DietPi to your SD card. Before ejecting it, open the `/boot` partition on your computer and edit `dietpi.txt`.

Set these specific values to ensure the Pi doesn't ask for user input:

```bash
AUTO_SETUP_AUTOMATED=1
AUTO_SETUP_GLOBAL_PASSWORD=your_irrigation_pass
AUTO_SETUP_INSTALL_SOFTWARE_ID=122,123  # 122=Node-RED, 123=Mosquitto
AUTO_SETUP_CUSTOM_SCRIPT_EXEC=0        # Tells DietPi to look for a local script

```

---

### 2. The "Magic" Automation Script

Create a new file in the `/boot` folder named **`Automation_Custom_Script.sh`**.

This script will run automatically after Node-RED is installed. It will download your "starter" irrigation flow and restart the service so it’s live immediately.

```bash
#!/bin/bash
# DietPi Post-Install Script for Smart Irrigation

# 1. Define the location of the flow file
FLOW_PATH="/mnt/dietpi_userdata/node-red/flows.json"

# 2. Wait for Node-RED directory to be created
mkdir -p /mnt/dietpi_userdata/node-red

# 3. Inject the Irrigation Starter Flow (JSON)
cat <<EOF > $FLOW_PATH
[{"id":"irrigation_flow","type":"tab","label":"Irrigation Control"},{"id":"relay1","type":"mqtt out","z":"irrigation_flow","name":"Zone 1 Valve","topic":"garden/zone1/command","broker":"localhost","x":450,"y":100,"wires":[]},{"id":"manual_btn","type":"ui_switch","z":"irrigation_flow","name":"Manual Water","label":"Zone 1","group":"main_group","order":1,"width":0,"height":0,"passthru":true,"topic":"","onvalue":"ON","offvalue":"OFF","x":200,"y":100,"wires":[["relay1"]]}]
EOF

# 4. Install the Dashboard nodes (essential for the UI)
cd /mnt/dietpi_userdata/node-red
npm install node-red-dashboard node-red-contrib-schedex

# 5. Set correct permissions and restart
chown -R nodered:nodered /mnt/dietpi_userdata/node-red
systemctl restart node-red

```

---

### 3. Deployment Summary

Once you insert the SD card and power up:

1. **DietPi** expands the disk and updates the OS.
2. It installs **Mosquitto** (MQTT) and **Node-RED**.
3. It runs your **Custom Script**, which:
* Places your pre-written `flows.json` into the Node-RED folder.
* Installs the **Dashboard** and **Scheduler** plugins automatically.
* Restarts Node-RED to load the new config.



### 4. Why this is perfect for the Model B

* **Zero User Input:** No need to connect a monitor or keyboard to the Pi.
* **Repeatability:** If your SD card ever fails, you just re-run this process and you are back online in 15 minutes.
* **Lightweight:** Since it’s all bash-scripted on a minimal OS, it preserves every megabyte of that 512MB RAM for the actual irrigation logic.

**Would you like me to provide the final "Production" ESPHome YAML for your 3 ESP32s so they are ready to talk to this newly automated hub?**
