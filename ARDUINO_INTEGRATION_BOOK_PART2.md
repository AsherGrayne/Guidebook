---

## 🌐 **Web Application Integration**

### **Configuration Setup:**

```python
# Arduino Integration Configuration
ARDUINO_SERVER_URL = "http://localhost:5001"  # Real Arduino server address
arduino_data_cache = []                       # Stores Arduino data in memory
arduino_connection_status = "disconnected"    # Tracks connection status

# Path to the simulated Arduino data file
SIMULATED_ARDUINO_DATA_PATH = os.path.join(os.path.dirname(os.path.dirname(__file__)), 
                                          'arduino_integration', 'latest_arduino_data.json')

# Store rolling window for live graphs
sim_data_history = {
    'temperature': deque(maxlen=100),  # Last 100 temperature readings
    'humidity': deque(maxlen=100),     # Last 100 humidity readings
    'voltage': deque(maxlen=100),      # Last 100 voltage readings
    'current': deque(maxlen=100),      # Last 100 current readings
    'time': deque(maxlen=100)          # Last 100 timestamps
}
```

### **Three Data Retrieval Methods:**

#### **Method 1: Real Arduino Data (Primary)**
```python
def fetch_arduino_data():
    """Fetch data from Arduino receiver"""
    global arduino_data_cache, arduino_connection_status
    try:
        response = requests.get(f"{ARDUINO_SERVER_URL}/api/arduino/data?limit=100", timeout=2)
        if response.status_code == 200:
            data = response.json()
            arduino_data_cache = data
            arduino_connection_status = "connected"
            return data
        else:
            arduino_connection_status = "error"
            return []
    except:
        arduino_connection_status = "disconnected"
        return []
```

#### **Method 2: Arduino Status Check**
```python
def get_arduino_status():
    """Get Arduino connection status"""
    global arduino_connection_status
    try:
        response = requests.get(f"{ARDUINO_SERVER_URL}/api/arduino/status", timeout=2)
        if response.status_code == 200:
            status = response.json()
            arduino_connection_status = "connected"
            return status
        else:
            arduino_connection_status = "error"
            return None
    except:
        arduino_connection_status = "disconnected"
        return None
```

#### **Method 3: Simulated Data (Fallback)**
```python
def read_simulated_arduino_data():
    """Read the latest simulated Arduino data from file."""
    try:
        with open(SIMULATED_ARDUINO_DATA_PATH, 'r') as f:
            data = json.load(f)
        return data
    except Exception:
        return None
```

### **How the Three Methods Work Together:**

| Method | Purpose | When Used |
|--------|---------|-----------|
| **Real Arduino** | Get data from actual sensors | When real Arduino is connected |
| **Status Check** | Check if Arduino is working | Every few seconds |
| **Simulated Data** | Get fake data for testing | When real Arduino is not available |

---

## 🔄 **Data Flow Process**

### **Main Callback Function:**

```python
@app.callback(
    [Output('arduino-temp-value', 'children'),
     Output('arduino-humidity-value', 'children'),
     Output('arduino-current-value', 'children'),
     Output('sim-temp-graph', 'figure'),
     Output('sim-humidity-graph', 'figure'),
     Output('sim-voltage-graph', 'figure'),
     Output('sim-current-graph', 'figure')],
    [Input('interval-component', 'n_intervals')]
)
def update_simulated_arduino_live(n):
    data = read_simulated_arduino_data()
    now = datetime.now()
    if data is not None:
        sim_data_history['temperature'].append(data.get('temperature'))
        sim_data_history['humidity'].append(data.get('humidity'))
        sim_data_history['voltage'].append(data.get('voltage'))
        sim_data_history['current'].append(data.get('current'))
        sim_data_history['time'].append(now)
    
    # Prepare figures
    def make_fig(y, ylabel):
        return {
            'data': [go.Scatter(
                x=list(sim_data_history['time']), 
                y=list(sim_data_history[y]), 
                mode='lines+markers', 
                line={'color': '#39ff14'}
            )],
            'layout': go.Layout(
                margin={'l': 40, 'r': 10, 't': 20, 'b': 40},
                xaxis={'title': 'Time', 'tickformat': '%H:%M:%S'},
                yaxis={'title': ylabel},
                template='plotly_dark',
                height=220
            )
        }
    
    temp_fig = make_fig('temperature', 'Temperature (°C)')
    humidity_fig = make_fig('humidity', 'Humidity (%)')
    voltage_fig = make_fig('voltage', 'Voltage (V)')
    current_fig = make_fig('current', 'Current (A)')
    
    # Live values
    temp_val = f"{sim_data_history['temperature'][-1] if sim_data_history['temperature'] else '--'}"
    humidity_val = f"{sim_data_history['humidity'][-1] if sim_data_history['humidity'] else '--'}"
    current_val = f"{sim_data_history['current'][-1] if sim_data_history['current'] else '--'}"
    
    return temp_val, humidity_val, current_val, temp_fig, humidity_fig, voltage_fig, current_fig
```

### **Step-by-Step Process:**

**1. Data Generation (simulated_arduino_output.py):**
```python
# Every 2 seconds:
temperature, humidity, voltage, current = generate_simulated_data()
data = {"temperature": temperature, "humidity": humidity, "voltage": voltage, "current": current}
# Write to latest_arduino_data.json
```

**2. Data Reading (app.py - update_simulated_arduino_live):**
```python
def update_simulated_arduino_live(n):
    data = read_simulated_arduino_data()  # Read from JSON file
    now = datetime.now()
    
    if data is not None:
        # Add to rolling history (keeps last 100 readings)
        sim_data_history['temperature'].append(data.get('temperature'))
        sim_data_history['humidity'].append(data.get('humidity'))
        sim_data_history['voltage'].append(data.get('voltage'))
        sim_data_history['current'].append(data.get('current'))
        sim_data_history['time'].append(now)
```

**3. Real-time Updates:**
```python
# This callback runs every 2 seconds (interval-component)
@app.callback(
    [Output('arduino-temp-value', 'children'),      # Live temperature display
     Output('arduino-humidity-value', 'children'),  # Live humidity display
     Output('arduino-current-value', 'children'),   # Live current display
     Output('sim-temp-graph', 'figure'),           # Temperature graph
     Output('sim-humidity-graph', 'figure'),       # Humidity graph
     Output('sim-voltage-graph', 'figure'),        # Voltage graph
     Output('sim-current-graph', 'figure')],       # Current graph
    [Input('interval-component', 'n_intervals')]
)
```

**4. Graph Creation:**
```python
def make_fig(y, ylabel):
    return {
        'data': [go.Scatter(
            x=list(sim_data_history['time']), 
            y=list(sim_data_history[y]), 
            mode='lines+markers', 
            line={'color': '#39ff14'}
        )],
        'layout': go.Layout(
            margin={'l': 40, 'r': 10, 't': 20, 'b': 40},
            xaxis={'title': 'Time', 'tickformat': '%H:%M:%S'},
            yaxis={'title': ylabel},
            template='plotly_dark',
            height=220
        )
    }
```

### **Data Flow Diagram:**

```
┌─────────────────────────┐    ┌─────────────────────────┐    ┌─────────────────────────┐
│   simulated_arduino_    │    │   latest_arduino_       │    │        app.py           │
│      output.py          │    │      data.json          │    │                        │
│                         │    │                         │    │                        │
│ • Generates random      │───▶│ • Stores latest         │───▶│ • Reads JSON file       │
│   sensor data           │    │   sensor readings       │    │ • Updates dashboard     │
│ • Runs every 2 seconds  │    │ • Updated every 2 sec   │    │ • Creates live graphs   │
│ • Writes to JSON file   │    │ • JSON format           │    │ • Shows real-time data  │
└─────────────────────────┘    └─────────────────────────┘    └─────────────────────────┘
```

---

## 🚀 **How to Run the System**

### **Step 1: Start the Arduino Simulator**

Open a terminal/command prompt and navigate to the project folder:

```bash
cd arduino_integration
python simulated_arduino_output.py
```

**What You'll See:**
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

### **Step 2: Start the Web Application**

Open another terminal/command prompt and navigate to the web folder:

```bash
cd web
python app.py
```

**What You'll See:**
```
Starting ENMOS Arduino Integration Dashboard on 0.0.0.0:8050
```

### **Step 3: Open the Dashboard**

Open your web browser and go to:
```
http://localhost:8050
```

### **Step 4: What You'll See**

1. **Arduino Connection Status**: Shows if Arduino is connected
2. **Live Sensor Values**: Real-time temperature, humidity, current, etc.
3. **Live Graphs**: Rolling graphs showing sensor data over time
4. **Data Tables**: Recent sensor readings with timestamps
5. **Anomaly Detection**: Analysis of sensor data for problems

### **Complete Setup Commands:**

```bash
# Terminal 1 - Start Arduino Simulator
cd Anomaly_Visualizer-main/arduino_integration
python simulated_arduino_output.py

# Terminal 2 - Start Web Application
cd Anomaly_Visualizer-main/web
python app.py

# Browser - Open Dashboard
# Go to: http://localhost:8050
```

---

## 🔧 **Troubleshooting**

### **Common Issues and Solutions:**

| Problem | Cause | Solution |
|---------|-------|----------|
| **"No Arduino data available"** | Simulator not running | Start `simulated_arduino_output.py` |
| **Dashboard not loading** | Web app not running | Start `app.py` in web folder |
| **No live updates** | Files not in correct location | Check file paths in code |
| **Graphs not showing** | No data in history | Wait 2-3 minutes for data to accumulate |
| **Import errors** | Missing Python packages | Install requirements: `pip install -r requirements.txt` |
| **File not found** | Wrong directory | Navigate to correct project folder |

### **Checking if Everything is Working:**

1. **Check the JSON file:**
   ```bash
   cat arduino_integration/latest_arduino_data.json
   ```
   Should show something like:
   ```json
   {"temperature": 26.8, "humidity": 62, "voltage": 2.452341, "current": 0.07234}
   ```

2. **Check the simulator output:**
   - Should print new data every 2 seconds
   - Should show realistic values

3. **Check the web dashboard:**
   - Should show live sensor values
   - Should update every 2 seconds
   - Should show green "Arduino Connected" status

### **Debugging Steps:**

1. **Verify File Structure:**
   ```
   Anomaly_Visualizer-main/
   ├── arduino_integration/
   │   ├── simulated_arduino_output.py
   │   └── latest_arduino_data.json
   └── web/
       └── app.py
   ```

2. **Check Python Installation:**
   ```bash
   python --version
   pip list
   ```

3. **Verify Dependencies:**
   ```bash
   pip install dash dash-bootstrap-components plotly pandas numpy requests
   ```

4. **Check File Permissions:**
   ```bash
   ls -la arduino_integration/
   ls -la web/
   ```

---

## 🔌 **Real Arduino Integration**

### **What You Need for Real Arduino:**

| Component | Purpose | Example Values |
|-----------|---------|----------------|
| **Arduino Board** | Main controller | Arduino Uno, Nano, Mega |
| **Temperature Sensor** | Measure temperature | DHT22, LM35 |
| **Humidity Sensor** | Measure humidity | DHT22, DHT11 |
| **Current Sensor** | Measure electrical current | ACS712 |
| **Voltage Sensor** | Measure voltage | Voltage divider |
| **Vibration Sensor** | Measure vibration | ADXL335 |
| **Pressure Sensor** | Measure pressure | BMP280 |

### **Arduino Code Example:**

```cpp
#include <DHT.h>
#include <Wire.h>
#include <Adafruit_Sensor.h>
#include <Adafruit_BMP280.h>

#define DHTPIN 2
#define DHTTYPE DHT22

DHT dht(DHTPIN, DHTTYPE);
Adafruit_BMP280 bmp;

void setup() {
  Serial.begin(9600);
  dht.begin();
  bmp.begin();
}

void loop() {
  float temperature = dht.readTemperature();
  float humidity = dht.readHumidity();
  float pressure = bmp.readPressure() / 100.0;
  
  // Read current sensor (ACS712)
  float current = analogRead(A0) * (5.0 / 1023.0) * 20.0; // 20A sensor
  
  // Create JSON data
  Serial.print("{\"temperature\":");
  Serial.print(temperature);
  Serial.print(",\"humidity\":");
  Serial.print(humidity);
  Serial.print(",\"pressure\":");
  Serial.print(pressure);
  Serial.print(",\"current\":");
  Serial.print(current);
  Serial.println("}");
  
  delay(2000); // Wait 2 seconds
}
```

### **Setting Up Real Arduino Server:**

You would need to create an Arduino data receiver server that:
1. Receives data from Arduino via serial/USB
2. Processes the data
3. Provides HTTP API endpoints
4. Runs on port 5001

### **Real Arduino vs Simulation:**

| Feature | Real Arduino | Simulation |
|---------|-------------|------------|
| **Data Source** | Physical sensors | Random generation |
| **Accuracy** | Real-world values | Approximate values |
| **Cost** | Hardware required | Free |
| **Setup Time** | Hours to days | Minutes |
| **Maintenance** | Hardware maintenance | No maintenance |
| **Reliability** | Depends on hardware | Always works |

---

## 📚 **Glossary**

| Term | Definition |
|------|------------|
| **Arduino** | Open-source electronics platform for building digital devices |
| **Sensor** | Device that detects and responds to input from the environment |
| **Simulation** | Imitation of real-world processes or systems |
| **JSON** | JavaScript Object Notation - a data format for storing information |
| **Callback** | Function that gets called automatically when something happens |
| **Real-time** | Processing data as it happens, without delay |
| **Anomaly** | Something that deviates from what is standard or expected |
| **Dashboard** | Visual display of important information |
| **API** | Application Programming Interface - way for programs to communicate |
| **Deque** | Double-ended queue - data structure for storing items |
| **HTTP** | HyperText Transfer Protocol - way for web applications to communicate |
| **Serial Communication** | Method for Arduino to send data to computer |
| **Machine Learning** | Artificial intelligence that learns from data |
| **Predictive Maintenance** | Scheduling maintenance based on sensor data |
| **IoT** | Internet of Things - connecting devices to the internet |

---

## 🎓 **Summary**

### **What You've Learned:**

1. **Arduino Integration**: How sensors connect to the web application
2. **Simulation**: How fake data is generated for testing
3. **Data Flow**: How information moves through the system
4. **Real-time Updates**: How the dashboard stays current
5. **Troubleshooting**: How to fix common problems

### **Key Concepts:**

- **Simulated Arduino** generates realistic sensor data
- **Web Application** reads data and displays it in real-time
- **Data History** keeps track of recent readings for graphs
- **Fallback System** ensures the app works even without real hardware
- **Real-time Updates** happen every 2 seconds automatically

### **System Architecture:**

```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   Arduino       │    │   Data          │    │   Web           │
│   Simulator     │───▶│   Storage       │───▶│   Dashboard     │
│                 │    │   (JSON)        │    │                 │
└─────────────────┘    └─────────────────┘    └─────────────────┘
```

### **Next Steps:**

1. **Run the system** using the instructions above
2. **Experiment** with different sensor values
3. **Explore** the dashboard features
4. **Consider** adding real Arduino hardware
5. **Customize** the system for your needs

### **Applications:**

- **Industrial Monitoring**: Watch factory equipment
- **Environmental Monitoring**: Track temperature and humidity
- **Energy Management**: Monitor power consumption
- **Predictive Maintenance**: Schedule repairs before breakdowns
- **Research**: Collect and analyze sensor data

---

## 📖 **Additional Resources**

### **Books:**
- "Arduino Cookbook" by Michael Margolis
- "Getting Started with Arduino" by Massimo Banzi
- "Python for Data Analysis" by Wes McKinney

### **Online Resources:**
- Arduino Official Website: https://www.arduino.cc/
- Dash Documentation: https://dash.plotly.com/
- Python Documentation: https://docs.python.org/

### **Communities:**
- Arduino Forum: https://forum.arduino.cc/
- Python Community: https://www.python.org/community/
- IoT Communities on Reddit and Discord

---

**🎉 Congratulations!** You now have a complete understanding of Arduino integration in the ENMOS Anomaly Visualization System. You can explain it to others, run the system, and even extend it with real hardware!

---

*This guidebook was created to help you understand and use the Arduino integration features of the ENMOS Anomaly Visualization System. Feel free to share this knowledge with others who are interested in IoT, sensor technology, and industrial monitoring.* 