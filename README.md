# Tesla Dynamic Charging Automation for Home Assistant (also Ford, Mercedes... any EV)

This project automates the charging current of an electric vehicle based on household power consumption.  
It works with **any EV (Tesla, Mercedes, Ford, BYD, etc.)** or **any EV charger** as long as it is integrated and can be controlled within Home Assistant.

This blueprint dynamically adjusts the charging rate, ensuring that the vehicle uses the **maximum available power** without exceeding the contracted limit.  
It also supports **different power limits for day and night** to optimize charging based on electricity tariffs.

**If you found this project helpful, please consider buying me a coffee!**

<div align="center">
  <a href='https://ko-fi.com/K3K819CO23' target='_blank'>
    <img height='36' style='border:0px;height:36px;' 
         src='https://storage.ko-fi.com/cdn/kofi6.png?v=6' border='0' 
         alt='Buy Me a Coffee at ko-fi' />
  </a>
</div>

---
## 🆕 **Latest Update: Version 2.2 Released!**
### ⭐ **What's New in v2.2**
#### **Optional Location Tracker**
- **Car/Person Location Tracker is now optional**: Leave the field empty during configuration to disable location-based checks entirely
- **Multi-vehicle support**: Ideal for households with more than one EV where tying regulation to a specific person/vehicle leads to incorrect behavior
- **Shared charger friendly**: Useful when the charger may serve cars not linked to the configured tracker (guests, family vehicles, etc.)
- **Backwards compatible**: Existing setups using the location tracker continue to work exactly as before

**No action required for existing users** - the previous behavior is preserved when a tracker entity is configured.

---

### 📋 **Previous Updates**

<details>
<summary><b>Version 2.1 - Case-Insensitive Charging Status Support</b></summary>

#### **Case-Insensitive Charging Status Support**
- **ESPhome Tesla BLE compatibility**: Now supports "Charging" (capital C) status from ESPhome Tesla BLE client
- **Multiple case format support**: Accepts `charging`, `Charging`, `CHARGING`, and other variations
- **Improved status detection**: Uses `| lower` filter for robust state comparison
- **Better integration compatibility**: Works seamlessly with different Home Assistant integrations and custom components

**No configuration changes required** - existing automations will automatically benefit from improved status detection.
</details>

<details>
<summary><b>Version 2.0 - Smart Start/Stop Control</b></summary>

#### **Smart Start/Stop Control**
- **Automatic charger control**: Instead of trying to set charging current to 0A (which many chargers can't handle), the automation now properly stops and starts charging
- **Prevents charger errors** caused by invalid amperage values
- **Seamless operation** with intelligent state management

#### **Fixed Power Calculation Logic**
- Resolved critical issues in power calculation that could cause incorrect decisions
- **No more oscillating** start/stop behavior
- **Accurate power management** ensures optimal charging

#### **Enhanced Stability**
- **Gradual amperage adjustment** prevents rapid changes that cause charger errors
- **Improved reliability** during power fluctuations
- **Better error handling** for edge cases

**Note for v2.0 users**: You must recreate your automation as new configuration options were added.
</details>

---
## 📥 Installation

This blueprint can be installed in two ways:

### **🔗 Direct Import via Home Assistant**
<a href="https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https://raw.githubusercontent.com/EDV11/electric-vehicle-ev-dynamic-charging-home-assistant-/main/Dynamic-EV-Charging-Automation.yaml" target="_blank" rel="noreferrer noopener"><img src="https://my.home-assistant.io/badges/blueprint_import.svg" alt="Open your Home Assistant instance and show the blueprint import dialog with a specific blueprint pre-filled." /></a>

### **📂 Manual Installation**
1. Open the blueprint file in this repository (https://raw.githubusercontent.com/EDV11/electric-vehicle-ev-dynamic-charging-home-assistant-/main/Dynamic-EV-Charging-Automation.yaml).
2. Copy the URL from your browser's address bar.
3. In Home Assistant, go to **Settings → Automations & Scenes → Blueprints**.
4. Click **"Import Blueprint"** in the bottom right corner.
5. Paste the URL and click **"Preview Blueprint"**.
6. Click **"Import Blueprint"** to finalize.

---

## 🔌 🚗 🔋 Tested Hardware  

This is some of the hardware I've used and tested with this automation:  

- [Shelly EM Energy Meter](https://amzlink.to/az0UFbhU9htEu)  
- Tesla car with [Tesla Fleet integration](https://www.home-assistant.io/integrations/tesla_fleet/).  
  - [Buy your Tesla with a discount through this link](https://ts.la/eduardo432919)  
- [Feyree Type 2 portable 7kW charger](https://s.click.aliexpress.com/e/_opguDAp)  
- [Feyree Type 2 Wallbox](https://s.click.aliexpress.com/e/_oEFZ75f)  

Just to be clear, if you can control your car from Home Assistant and have access to its sensors, you can use this automation without a connected charger. But by controlling the charger you make sure future compatibility (and be able to charge different cars)

---

## 🔧 Requirements

Before using this blueprint, you need the following:

- **Total Power Consumption Sensor** (e.g., <a href="https://amzlink.to/az0UFbhU9htEu" target=_blank>Shelly EM Gen3</a> is recommended if you don't have one. You can get it from that link).
- **Your EV or Charger integrated into Home Assistant**, exposing the necessary sensors. I use this <a href="https://s.click.aliexpress.com/e/_opguDAp" target=_blank>Feyree portable charger</a>, I haven't tested with any other.
- **Basic information about your electricity contract**, including:
  - Grid voltage (e.g., 230V for EU, 120V for the US).
  - Maximum grid power limits (contracted power).
  - Day and night tariff periods (if applicable).

---

## ⚙️ Configuration

The following parameters need to be configured:

### **Basic Settings**
- **Charging interval** (1, 3, or 5 minutes).
- **🆕 A device tracker (optional in v2.2)** – Preferably the car, to ensure the automation runs only when the EV is at home. **Leave empty to disable** the location check entirely (useful for shared chargers or multi-vehicle households).
- **A charging status sensor** to ensure changes are only made when the EV is actively charging.
- **Total power consumption sensor** (measured in watts).

### **Power Management**
- **Maximum Grid Power (Day)** – Set to **0** if you only want to charge at night.
- **Maximum Grid Power (Night)**.
- **Grid voltage**.
- **Maximum charging amperage** – The system will never exceed this limit, even if extra power is available.
- **Minimum charging amperage** – The system will stop charging if power availability drops below this threshold.

### **Charger Control**
- **Charging current control entity** – The EV or charger must support dynamic amperage control.
- **EV Charging Toggle (Start/Stop)** – A `select` entity that controls your charger's start/stop state with options like:
  - `"Start charging"`
  - `"Stop charging"`
  - `"Waiting for command"`

### **Time Periods**
- **Day-time period settings** (Define when "day" mode applies, typically when electricity is more expensive or power is limited).

---

## 🚀 How It Works

The automation runs every **1, 3, or 5 minutes**, depending on the selected interval.  
It **only** adjusts the charging rate if the EV is **charging** and (if configured) **at home**.  

### **Intelligent Power Management**
1. **Monitors** total household power consumption
2. **Calculates** available power for EV charging
3. **Adjusts** charging current gradually (1A steps with delays)
4. **Stops charging** when insufficient power (instead of attempting invalid 0A)
5. **Restarts automatically** when power becomes available

### **Dynamic Load Adjustment**
The automation continuously monitors household power usage and dynamically **adjusts the charging current** to avoid exceeding the contracted power limit.  
If a high-power appliance (e.g., oven, stove, heating) is turned on, the charging rate will **automatically decrease** to prevent overloading.  
Once power consumption drops, the EV will **resume charging at the maximum allowed rate**.

### **Day & Night Power Limits**
Many electricity contracts offer **cheaper power at night**, along with **higher power limits**.  
This blueprint allows setting **different power limits for day and night**, ensuring that charging is optimized based on your tariff structure.

### **Safety Features**
- **Maximum/minimum amperage limits**
- **Gradual adjustments** to prevent charger errors
- **Optional location check** – when configured, only operates when car is home and plugged in
- **Respects charger capabilities**

---

## 🔍 **Compatibility**

**Works with any EV charger that has:**

- ✅ Amperage control via Home Assistant (`number` entity)
- ✅ Start/stop control via Home Assistant (`select` entity) 
- ✅ Charging state sensor (`sensor` entity)

---

## 💡 **Tips for Best Results**

1. **Set realistic min/max amperage** based on your electrical system
2. **Monitor for the first few days** to ensure stable operation
3. **Adjust time periods** to match your electricity tariff
4. **Use shorter intervals** (1 min) for faster response to power changes
5. **Disable the location tracker** (leave empty) if you have multiple EVs or share the charger with other vehicles

---

## ❓ Reporting Issues & Community Discussion

If you encounter issues or have suggestions for improvements, please check the **Home Assistant Forum thread** for support.  
[https://community.home-assistant.io/t/dynamic-ev-charging-automation/846232](https://community.home-assistant.io/t/dynamic-ev-charging-automation/846232)

---

## 🙏 **Feedback & Support**

This blueprint has been tested extensively, but every setup is different. Please:

- ⭐ **Star the repository** if you find it useful
- 🐛 **Report issues** via forum
- 💬 **Share your experience** in the comments
- 🔄 **Suggest improvements** for future versions

---

## 📜 License

This project is released under the **MIT License**, which means you are free to **download, modify, and share it** as you like.

If you prefer **Creative Commons**, the most suitable option would be **CC BY-SA 4.0** (Attribution-ShareAlike), which allows modifications as long as credit is given.

Would you like to use **MIT**, **CC BY-SA 4.0**, or another license? Let me know, and I'll finalize this section accordingly!
