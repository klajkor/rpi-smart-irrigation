# Smart irragation control with Raspberry Pi Model B

For a 2012-era **Raspberry Pi Model B** with only 512MB of RAM, **Node-RED** is significantly more lightweight and reliable than Home Assistant.

Here is the step-by-step plan to turn that Pi into a lean, mean irrigation brain.

---

## 1. Prepare the Pi (The "Lean" Install)

Don't install a desktop environment; it wastes precious RAM.

1. Flash **Raspberry Pi OS Lite** (64-bit or 32-bit) using the Raspberry Pi Imager.
2. Enable **SSH** in the settings so you can manage it from your computer.
3. Boot the Pi and run the official Node-RED install script:
```bash
bash <(curl -sL https://raw.githubusercontent.com/node-red/linux-installers/master/deb/update-nodejs-and-nodered)

```


4. **Crucial Step for Old Hardware:** When asked if you want to install Pi-specific nodes, say **Yes**.
5. Set Node-RED to start automatically on boot:
```bash
sudo systemctl enable nodered.service

```


6. Start it manually for now: `node-red-start`.

---

## 2. Install the MQTT Broker

Since you have 3 ESP32s, they need a way to "talk" to the Pi. **Mosquitto** is the industry standard and uses almost zero resources.

1. Install it: `sudo apt install mosquitto mosquitto-clients`
2. Enable it: `sudo systemctl enable mosquitto`

---

## 3. Create the Irrigation Dashboard

Open a browser on your laptop and go to `http://<your-pi-ip>:1880`. You’ll see the Node-RED flow editor.

### Required "Nodes" (Install via Manage Palette):

* **node-red-dashboard**: This creates the UI (buttons/sliders) for your phone.
* **node-red-contrib-ui-schedex**: A visual scheduler specifically for simple "On/Off" times.

### The Flow Logic:

1. **Manual Trigger:** A `UI Button` node connects to an `MQTT Out` node. When you press "Water Now," it sends a message like `garden/zone1/command` with the payload `ON`.
2. **The Schedule:** A `Schedex` node lets you set "Start at 06:00" and "Stop at 06:15."
3. **Status Updates:** An `MQTT In` node listens for `garden/zone1/status`. This links to a `UI Text` or `LED` node so you can see if the valve is actually open.

---

## 4. Flash the ESP32s (ESPHome)

Since you are now using Node-RED + MQTT (no Home Assistant), your ESPHome configuration needs a small tweak to use the **MQTT Client** instead of the native API.

**Example snippet for your ESP32:**

```yaml
mqtt:
  broker: 192.168.1.XX # Your Pi's IP
  topic_prefix: garden/zone1

switch:
  - platform: gpio
    pin: 21
    name: "Zone 1 Valve"
    state_topic: "garden/zone1/status"
    command_topic: "garden/zone1/command"
    payload_on: "ON"
    payload_off: "OFF"

```

---

## 5. Why this works for your Pi Model B

* **RAM Management:** Node-RED runs in about 150MB. This leaves plenty of "breathing room" for the OS and MQTT.
* **Decoupling:** If the Pi crashes, your ESP32s can be programmed (via ESPHome's `on_boot` rules) to fail-safe and shut the water off regardless.
* **Responsiveness:** Without the massive Home Assistant database overhead, your dashboard will load instantly on your phone.

**Would you like me to provide the full Node-RED "Flow JSON" that you can simply copy and paste to get your first dashboard started?**
