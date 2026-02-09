# Starter flow for Node-RED

To get you up and running quickly, I have prepared a **starter flow**. You can import this directly into Node-RED to create a functional dashboard with a manual override and a basic schedule.

### 1. The "Flow" Code

Copy the JSON block below. In Node-RED, go to the **Menu (top right) > Import**, paste this code, and click **Import**.

```json
[{"id":"bc51643c.966458","type":"mqtt out","z":"a1b2c3d4.e5f6g7","name":"Send to ESP32","topic":"garden/zone1/command","qos":"","retain":"","broker":"mqtt_broker_id","x":650,"y":120,"wires":[]},{"id":"1a2b3c4d.5e6f7g","type":"ui_switch","z":"a1b2c3d4.e5f6g7","name":"Manual Override","label":"Zone 1 Water","group":"ui_group_id","order":1,"width":0,"height":0,"passthru":true,"decouple":"false","topic":"","style":"","onvalue":"ON","onvalueType":"str","oncolor":"","onicon":"","offvalue":"OFF","offvalueType":"str","offcolor":"","officon":"","x":410,"y":120,"wires":[["bc51643c.966458"]]},{"id":"sched123","type":"schedex","z":"a1b2c3d4.e5f6g7","name":"Morning Schedule","passthrough":false,"onetime":false,"ontime":"06:00","offtime":"06:15","mon":true,"tue":true,"wed":true,"thu":true,"fri":true,"sat":true,"sun":true,"x":180,"y":120,"wires":[["1a2b3c4d.5e6f7g"]]}]

```

---

### 2. Physical Setup & Safety

Since you are dealing with water and electricity, your hardware assembly is the most critical part. Your ESP32 will trigger a relay, which acts as the gatekeeper for the 24V AC power required by the solenoid valves.

**Safety Note:** Never power the solenoid valves directly from the ESP32 or the Raspberry Pi. You must use an external **24V AC transformer** (standard irrigation power) and a **Relay Module** to bridge the two systems.

---

### 3. Component List (The "Shopping List")

Since you already have the Pi and ESP32s, here is exactly what else you need to buy:

| Component | Purpose | Approx. Cost |
| --- | --- | --- |
| **8-Channel Relay Module** | Allows the ESP32 to switch the 24V power. | $8 - $12 |
| **24V AC Power Adapter** | Standard power source for irrigation valves. | $15 - $25 |
| **Solenoid Valves** | The actual water valves (e.g., Orbit 57100). | $15 each |
| **DC-DC Buck Converter** | To drop 24V (or 12V) down to 5V to power the ESP32. | $3 - $5 |
| **Project Box (IP65)** | Waterproof housing for the electronics. | $10 - $20 |

---

### 4. Next Steps for Reliability

1. **Fixed IP Addresses:** Log into your router and assign a static IP to your Raspberry Pi. If the Pi's IP changes, your ESP32s won't be able to find the MQTT broker.
2. **The "Watchdog" Logic:** Inside the ESP32 code (ESPHome), add a `delayed_off` filter. This ensures that even if the Pi crashes or the Wi-Fi drops while the water is running, the ESP32 will automatically shut the valve off after 20 minutes as a safety backup.

