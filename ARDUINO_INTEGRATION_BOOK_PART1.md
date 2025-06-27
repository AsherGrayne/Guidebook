# 🚀 Arduino Integration Guidebook
## Complete Tutorial for ENMOS Anomaly Visualization System

---

## 📋 **Table of Contents**

1. [Introduction](#introduction)
2. [Arduino Integration Overview](#arduino-integration-overview)
3. [Folder Structure](#folder-structure)
4. [Simulated Arduino Output](#simulated-arduino-output)
5. [Web Application Integration](#web-application-integration)
6. [Data Flow Process](#data-flow-process)
7. [How to Run the System](#how-to-run-the-system)
8. [Troubleshooting](#troubleshooting)
9. [Real Arduino Integration](#real-arduino-integration)
10. [Glossary](#glossary)

---

## 🎯 **Introduction**

Welcome to the Arduino Integration Guidebook! This comprehensive guide will teach you everything about how the ENMOS Anomaly Visualization System connects with Arduino sensors to monitor industrial equipment in real-time.

### **What You'll Learn:**
- How Arduino sensors work with the system
- How simulated data is generated and used
- How the web dashboard displays real-time sensor data
- How to set up and run the system
- How to integrate real Arduino hardware

### **Prerequisites:**
- Basic understanding of Python
- Familiarity with web applications
- Interest in IoT and sensor technology

### **System Overview:**
The ENMOS system combines:
- **Arduino Sensors**: Collect real-time data from equipment
- **Data Processing**: Analyze sensor readings for anomalies
- **Web Dashboard**: Display results in an interactive interface
- **Machine Learning**: Predict maintenance needs

---

## 🔧 **Arduino Integration Overview**

### **What is Arduino Integration?**
Arduino integration means connecting physical sensors (like temperature, humidity, current sensors) to your computer so that the ENMOS system can read and analyze the data in real-time.

### **Why Use Arduino?**
- **Real-time Monitoring**: Get live data from actual sensors
- **Industrial Applications**: Monitor machinery, equipment, and processes
- **Anomaly Detection**: Identify problems before they become serious
- **Predictive Maintenance**: Schedule maintenance based on sensor data

### **How It Works:**
```
Arduino Sensors → Data Collection → Web Dashboard → Analysis & Alerts
```

### **Types of Sensors Used:**
| Sensor Type | Purpose | Example Values |
|-------------|---------|----------------|
| **Temperature** | Monitor equipment heat | 24-30°C |
| **Humidity** | Check environmental conditions | 55-70% |
| **Current** | Monitor electrical consumption | 0.05-0.09A |
| **Voltage** | Check power supply | 2.3-2.6V |
| **Vibration** | Detect mechanical issues | 0.001-0.005g |
| **Pressure** | Monitor fluid systems | 1000-1100 hPa |

---

## 📁 **Folder Structure**

### **Arduino Integration Folder Contents:**

The `arduino_integration` folder contains:

| File | Purpose | Description |
|------|---------|-------------|
| `simulated_arduino_output.py` | **Arduino Simulator** | Generates fake sensor data like a real Arduino would |
| `latest_arduino_data.json` | **Data Storage** | Stores the most recent sensor readings |

### **File Locations:**
```
Anomaly_Visualizer-main/
├── arduino_integration/
│   ├── simulated_arduino_output.py
│   └── latest_arduino_data.json
├── web/
│   └── app.py
└── [other folders...]
```

### **What Each File Does:**

**`simulated_arduino_output.py`**
- This is like having a "fake Arduino" that generates realistic sensor data
- Runs continuously and updates data every 2 seconds
- Creates data that looks like it came from real sensors
- Writes data to a JSON file for the web app to read

**`latest_arduino_data.json`**
- This is a data file that stores the latest sensor readings
- Gets updated every 2 seconds with new values
- The web application reads this file to display data
- Contains sensor values in JSON format

---

## 🤖 **Simulated Arduino Output**

### **Complete Code Analysis:**

```python
import time
import random
import json
import os

def generate_simulated_data():
    temperature = round(random.uniform(24, 30), 1)
    humidity = random.randint(55, 70)
    voltage = round(random.uniform(2.3, 2.6), 6)
    current = round(random.uniform(0.05, 0.09), 5)
    return temperature, humidity, voltage, current

# Path to write the latest data for dashboard
DATA_PATH = os.path.join(os.path.dirname(__file__), 'latest_arduino_data.json')

while True:
    temperature, humidity, voltage, current = generate_simulated_data()
    print(f"Vout: {voltage:.2f} | Raw current: {current:.2f}")
    print("{")
    print(f'  "temperature": {temperature},')
    print(f'  "humidity": {humidity},')
    print(f'  "voltage": {voltage},')
    print(f'  "current": {current}')
    print("}")
    print()
    # Write to file for dashboard
    data = {
        "temperature": temperature,
        "humidity": humidity,
        "voltage": voltage,
        "current": current
    }
    with open(DATA_PATH, "w") as f:
        json.dump(data, f)
    time.sleep(2)
```

### **Detailed Breakdown:**

#### **1. Imports (Lines 1-4):**
```python
import time      # For delays and timing
import random    # For generating random sensor values
import json      # For creating JSON data format
import os        # For file path operations
```

#### **2. Data Generation Function (Lines 6-10):**
```python
def generate_simulated_data():
    temperature = round(random.uniform(24, 30), 1)    # Random temperature: 24-30°C
    humidity = random.randint(55, 70)                 # Random humidity: 55-70%
    voltage = round(random.uniform(2.3, 2.6), 6)      # Random voltage: 2.3-2.6V
    current = round(random.uniform(0.05, 0.09), 5)    # Random current: 0.05-0.09A
    return temperature, humidity, voltage, current
```

**What This Does:**
- Creates realistic sensor values that change randomly
- Temperature varies between 24°C and 30°C (room temperature range)
- Humidity varies between 55% and 70% (normal indoor humidity)
- Voltage varies between 2.3V and 2.6V (typical sensor output)
- Current varies between 0.05A and 0.09A (low current range)

#### **3. File Path Setup (Line 12):**
```python
DATA_PATH = os.path.join(os.path.dirname(__file__), 'latest_arduino_data.json')
```

**What This Does:**
- Creates the full path to the data file
- `os.path.dirname(__file__)` gets the current folder
- Points to `latest_arduino_data.json` in the same folder

#### **4. Main Loop (Lines 14-35):**
```python
while True:  # Run forever
    # Generate new sensor data
    temperature, humidity, voltage, current = generate_simulated_data()
    
    # Print to console (for debugging)
    print(f"Vout: {voltage:.2f} | Raw current: {current:.2f}")
    print("{")
    print(f'  "temperature": {temperature},')
    print(f'  "humidity": {humidity},')
    print(f'  "voltage": {voltage},')
    print(f'  "current": {current}')
    print("}")
    print()
    
    # Create JSON data structure
    data = {
        "temperature": temperature,
        "humidity": humidity,
        "voltage": voltage,
        "current": current
    }
    
    # Write to file for the web app to read
    with open(DATA_PATH, "w") as f:
        json.dump(data, f)
    
    time.sleep(2)  # Wait 2 seconds before next reading
```

**What This Does:**
1. **Generates Data**: Creates new sensor values
2. **Prints to Console**: Shows the data (for debugging)
3. **Creates JSON**: Formats data as JSON
4. **Writes to File**: Saves data to `latest_arduino_data.json`
5. **Waits**: Pauses for 2 seconds before repeating

### **Sample Output:**
```
Vout: 2.45 | Raw current: 0.07
{
  "temperature": 26.8,
  "humidity": 62,
  "voltage": 2.452341,
  "current": 0.07234
}

Vout: 2.38 | Raw current: 0.06
{
  "temperature": 25.3,
  "humidity": 58,
  "voltage": 2.384567,
  "current": 0.06123
}
```

### **Why Use Simulation?**

**Benefits:**
- **No Hardware Required**: Test the system without buying sensors
- **Consistent Data**: Always have data to work with
- **Development**: Build and test features quickly
- **Demonstration**: Show how the system works to others

**Real vs Simulated:**
| Aspect | Real Arduino | Simulated Arduino |
|--------|-------------|-------------------|
| **Data Source** | Actual sensors | Random numbers |
| **Cost** | Hardware required | Free |
| **Reliability** | Depends on sensors | Always works |
| **Realism** | Real-world conditions | Controlled values | 