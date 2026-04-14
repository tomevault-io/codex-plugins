# Flotation Project - AI Coding Assistant Instructions

## Project Overview
Automated reagent dosing control system for small-to-medium scale gold mines in Zimbabwe. This is a **Raspberry Pi 5-based** vision control system that uses real-time froth monitoring to optimize frother dosage in flotation cells.

**Core Mission**: Maintain optimal froth characteristics through closed-loop PI control and anomaly detection, reducing reagent waste and improving gold recovery efficiency.

## System Architecture

### Hardware Stack
- **Controller**: Raspberry Pi 5 (quad-core ARM, 4-8GB RAM)
- **Vision**: 1080p USB webcam + USB-powered LED ring light (always on)
- **Actuation**: 
  - Frother pump (peristaltic) - GPIO 12 PWM
  - Agitator motor - GPIO 13 PWM
  - Air pump - GPIO 14 PWM
  - Feed pump - GPIO 15 PWM
- **Flotation Cell**: Acrylic cell with 4 actuators (frother, feed, air, agitator)
- **Safety**: Hard-wired emergency stop on GPIO 22 (E-STOP)

### Software Architecture
```
┌─────────────────────────────────────────────────┐
│  Web Dashboard (HTML/CSS/JS + WebSockets)      │
│  - Real-time video stream                       │
│  - Metrics visualization                        │
│  - Manual override controls                     │
└──────────────────┬──────────────────────────────┘
                   │
┌──────────────────▼──────────────────────────────┐
│  FastAPI Backend (Python 3)                     │
│  - REST endpoints + WebSocket server            │
│  - Data aggregation and state management        │
└──────────────────┬──────────────────────────────┘
                   │
     ┌─────────────┼─────────────┐
     │             │             │
┌────▼────┐  ┌─────▼─────┐  ┌───▼────────┐
│ Vision  │  │ Control   │  │  Anomaly   │
│ Module  │  │ Module    │  │  Detection │
│ OpenCV  │  │ PI Loop   │  │ IsolForest │
└─────────┘  └───────────┘  └────────────┘
```

### Technology Stack
- **Backend**: Python 3.11+, FastAPI, uvicorn
- **Vision Processing**: OpenCV (cv2), NumPy
- **Machine Learning**: scikit-learn (Isolation Forest)
- **Frontend**: Vanilla HTML/CSS/JS with Chart.js for visualization
- **Hardware Control**: lgpio (Raspberry Pi 5 native GPIO library, PWM control)
- **Data Storage**: SQLite (limited by RPi storage constraints)
- **Real-time Communication**: WebSockets (Server-Sent Events as fallback)

## Project Structure (Recommended)
```
/flotation_project/
├── config/
│   ├── camera_config.json      # Camera settings, resolution, FPS
│   ├── control_config.json     # PI gains, setpoints, pump parameters
│   └── system_config.json      # General system settings
├── src/
│   ├── vision/
│   │   ├── camera.py           # USB camera interface
│   │   ├── preprocessor.py     # Image enhancement, noise reduction
│   │   ├── bubble_detector.py  # Bubble segmentation and analysis
│   │   └── froth_analyzer.py   # Aggregate froth metrics
│   ├── control/
│   │   ├── pi_controller.py    # PI control loop implementation
│   │   ├── pump_driver.py      # Peristaltic pump PWM control
│   │   └── safety.py           # E-STOP handling, limits
│   ├── ml/
│   │   ├── anomaly_detector.py # Isolation Forest implementation
│   │   └── feature_extractor.py # Extract features from froth data
│   ├── api/
│   │   ├── main.py             # FastAPI app entry point
│   │   ├── routes.py           # REST endpoints
│   │   └── websocket.py        # WebSocket handlers for streaming
│   └── utils/
│       ├── logger.py           # Logging configuration
│       └── data_manager.py     # SQLite interface, data retention
├── dashboard/
│   ├── index.html              # Main dashboard page
│   ├── css/
│   │   └── styles.css
│   └── js/
│       ├── app.js              # Dashboard logic
│       ├── video_stream.js     # WebSocket video handling
│       └── charts.js           # Chart.js visualizations
├── tests/
│   ├── test_vision.py
│   ├── test_control.py
│   └── test_api.py
├── scripts/
│   ├── calibrate_camera.py     # Camera calibration utility
│   ├── tune_pi.py              # PI tuning assistant
│   └── simulate_froth.py       # Testing without hardware
├── requirements.txt
├── setup.sh                    # RPi deployment script
└── run.py                      # Main application entry
```

## Implementation Guidance

### 1. Vision Processing Pipeline (src/vision/)

**Image Preprocessing Strategy**:
```python
# preprocessor.py - Optimize for RPi performance
1. Convert to grayscale immediately (reduce 3x data)
2. Apply Gaussian blur (kernel=5x5) to reduce noise
3. Use adaptive thresholding (ADAPTIVE_THRESH_GAUSSIAN_C)
4. Morphological operations: opening -> closing (remove artifacts)
5. Resize to 640x480 if 1080p causes lag (balance quality vs speed)
```

**Bubble Detection Algorithm**:
- Use **watershed segmentation** or **contour detection** (cv2.findContours)
- Filter contours by area: `MIN_BUBBLE_AREA = 50px²`, `MAX_BUBBLE_AREA = 5000px²`
- Calculate metrics: count, mean area, size distribution (std dev)
- Use cv2.minEnclosingCircle() for bubble circularity validation
- **Performance**: Process every 2nd or 3rd frame if CPU > 70%

**Key Metrics to Extract**:
```python
# froth_analyzer.py
froth_metrics = {
    'bubble_count': int,
    'avg_bubble_size': float,  # pixels²
    'size_std_dev': float,
    'froth_stability': float,  # 0-1 score based on variance
    'coverage_ratio': float,   # bubble area / total area
    'timestamp': datetime
}
```

### 2. PI Controller Implementation (src/control/)

**Control Loop Architecture**:
```python
# pi_controller.py
class PIController:
    def __init__(self, kp=1.0, ki=0.1, setpoint=100):
        self.kp = kp  # Proportional gain (start conservative)
        self.ki = ki  # Integral gain (prevent windup!)
        self.setpoint = setpoint  # Target bubble count
        self.integral = 0.0
        self.last_error = 0.0
        
    def update(self, measured_value, dt):
        error = self.setpoint - measured_value
        self.integral += error * dt
        
        # Anti-windup: clamp integral term
        self.integral = np.clip(self.integral, -50, 50)
        
        output = (self.kp * error) + (self.ki * self.integral)
        return np.clip(output, 0, 100)  # PWM duty cycle 0-100%
```

**Tuning Strategy** (scripts/tune_pi.py):
- Start with Kp=0.5, Ki=0.05 (avoid oscillations)
- Use Ziegler-Nichols method or manual step response
- Typical loop time: 1-5 seconds (froth dynamics are slow)
- Log setpoint vs actual for offline tuning analysis

**Pump Control** (src/control/pump_driver.py):
```python
import lgpio  # Raspberry Pi 5 native GPIO library

# GPIO Pin Assignments
FROTHER_PUMP_PIN = 12  # Hardware PWM (PWM0)
AGITATOR_PIN = 13      # Hardware PWM (PWM1)
AIR_PUMP_PIN = 14      # Software PWM
FEED_PUMP_PIN = 15     # Software PWM

# Open GPIO chip (Raspberry Pi 5)
chip = lgpio.gpiochip_open(0)

# Initialize PWM for all devices (1kHz, 0% duty cycle)
for pin in [FROTHER_PUMP_PIN, AGITATOR_PIN, AIR_PUMP_PIN, FEED_PUMP_PIN]:
    lgpio.tx_pwm(chip, pin, 1000, 0)  # frequency=1kHz, duty_cycle=0%

# Control frother pump (example)
lgpio.tx_pwm(chip, FROTHER_PUMP_PIN, 1000, duty_cycle)  # duty_cycle 0-100%
```

### 3. Anomaly Detection (src/ml/)

**Isolation Forest Setup**:
```python
# anomaly_detector.py - Lightweight for RPi
from sklearn.ensemble import IsolationForest

class FrothAnomalyDetector:
    def __init__(self, contamination=0.1):
        self.model = IsolationForest(
            contamination=contamination,  # Expected anomaly rate
            max_samples=256,  # Small for RPi memory
            n_estimators=50,  # Balance accuracy vs speed
            random_state=42
        )
        self.is_trained = False
    
    def train(self, normal_data):
        # Train on 1-2 hours of stable operation
        # Features: [bubble_count, avg_size, std_dev, coverage]
        self.model.fit(normal_data)
        self.is_trained = True
    
    def predict(self, features):
        if not self.is_trained:
            return 1  # Normal until trained
        return self.model.predict([features])[0]  # 1=normal, -1=anomaly
```

**Feature Engineering**:
- Use rolling statistics (last 10 readings) to smooth noise
- Include temporal features: rate of change, variance over 1min
- Normalize features with StandardScaler before training

### 4. Web Dashboard Architecture (dashboard/)

**Real-time Data Streaming**:
```javascript
// video_stream.js - WebSocket for video + metrics
const ws = new WebSocket('ws://raspberrypi.local:8000/ws');

ws.onmessage = (event) => {
    const data = JSON.parse(event.data);
    
    if (data.type === 'frame') {
        updateVideoFrame(data.image);  // Base64 JPEG
    } else if (data.type === 'metrics') {
        updateMetrics(data.metrics);
        updateCharts(data.metrics);
    } else if (data.type === 'anomaly') {
        showAlert(data.message);
    }
};
```

**Dashboard Components**:
1. **Video Feed**: <canvas> or <img> updated via WebSocket (JPEG @ 10-15 FPS)
2. **Metrics Panel**: Real-time bubble count, size, pump duty cycle
3. **Chart.js Graphs**: Time-series plots (last 5 minutes, circular buffer)
4. **Control Panel**: Setpoint slider, PI gain inputs, manual pump override
5. **Status Indicators**: System health, anomaly alerts, E-STOP status

**Network Access**:
- Dashboard accessible from any device on local network
- FastAPI binds to `0.0.0.0:8000` (all interfaces)
- WebSocket connections supported across network
- CORS enabled for cross-origin requests
- Recommended: Set static IP on Raspberry Pi for consistent access

**Data Retention Strategy**:
```python
# data_manager.py - SQLite with automatic cleanup
import sqlite3
from datetime import datetime, timedelta

class DataManager:
    def __init__(self, db_path='flotation.db', retention_days=7):
        self.conn = sqlite3.connect(db_path)
        self.retention_days = retention_days
        self.create_tables()
    
    def cleanup_old_data(self):
        # Run daily - keep only last 7 days
        cutoff = datetime.now() - timedelta(days=self.retention_days)
        self.conn.execute(
            "DELETE FROM metrics WHERE timestamp < ?", (cutoff,)
        )
        self.conn.execute("VACUUM")  # Reclaim space
```

### 5. Performance Optimization for RPi 5

**Critical Optimizations**:
1. **Use cv2.UMat** for OpenCV GPU acceleration (if VideoCore VII supports)
2. **Multiprocessing**: Separate processes for vision, control, API
   ```python
   # run.py
   from multiprocessing import Process, Queue
   vision_queue = Queue(maxsize=2)  # Bounded queue
   p1 = Process(target=vision_loop, args=(vision_queue,))
   p2 = Process(target=control_loop, args=(vision_queue,))
   ```
3. **Frame Skipping**: Process every Nth frame if CPU > 80%
4. **Image Compression**: Send JPEG (quality=70) to dashboard, not PNG
5. **Reduce WebSocket Frequency**: 10 FPS video, 1 Hz metrics updates
6. **NumPy Optimization**: Use in-place operations (arr += 1 vs arr = arr + 1)

**Resource Monitoring**:
```python
# utils/logger.py - Track RPi health
import psutil

def log_system_stats():
    cpu = psutil.cpu_percent(interval=1)
    memory = psutil.virtual_memory().percent
    temp = psutil.sensors_temperatures()['cpu_thermal'][0].current
    
    if cpu > 85 or memory > 90 or temp > 75:
        logging.warning(f"High load: CPU={cpu}% MEM={memory}% TEMP={temp}°C")
```

### 6. Error Handling & Safety

**Hardware Failure Handling**:
```python
# safety.py - Fail-safe mechanisms
class SafetyManager:
    def __init__(self):
        self.estop_triggered = False
        self.watchdog_timeout = 5  # seconds
        self.last_vision_update = time.time()
    
    def check_watchdog(self):
        if time.time() - self.last_vision_update > self.watchdog_timeout:
            logging.critical("Vision system timeout - stopping pump")
            self.emergency_stop()
    
    def emergency_stop(self):
        # Immediately stop pump, close valves
        pump_driver.set_duty_cycle(0)
        self.estop_triggered = True
        send_alert("Emergency stop activated")
```

**Camera Disconnect Recovery**:
```python
# camera.py
def robust_camera_init(device_id=0, max_retries=5):
    for attempt in range(max_retries):
        cap = cv2.VideoCapture(device_id)
        if cap.isOpened():
            return cap
        logging.warning(f"Camera init failed, retry {attempt+1}/{max_retries}")
        time.sleep(2)
    raise RuntimeError("Camera initialization failed")
```

### 7. Deployment Configuration (RPi)

**setup.sh - System Preparation**:
```bash
#!/bin/bash
# Enable camera, I2C, SPI
sudo raspi-config nonint do_camera 0
sudo raspi-config nonint do_i2c 0

# Install system dependencies
sudo apt update
sudo apt install -y python3-opencv python3-lgpio python3-rpi-lgpio libopenblas-dev

# Python dependencies
pip3 install -r requirements.txt --break-system-packages

# Test lgpio installation
python3 -c "import lgpio; chip=lgpio.gpiochip_open(0); print('lgpio OK'); lgpio.gpiochip_close(chip)"

# Create systemd service
sudo cp flotation.service /etc/systemd/system/
sudo systemctl enable flotation.service
```

**systemd Service** (flotation.service):
```ini
[Unit]
Description=Flotation Control System
After=network.target

[Service]
Type=simple
User=pi
WorkingDirectory=/home/pi/flotation_project
ExecStart=/usr/bin/python3 run.py
Restart=always
RestartSec=10

[Install]
WantedBy=multi-user.target
```

**requirements.txt**:
```
fastapi==0.104.0
uvicorn[standard]==0.24.0
opencv-python==4.8.1.78
numpy==1.24.3
scikit-learn==1.3.2
psutil==5.9.6
python-multipart==0.0.6
websockets==12.0
# Note: lgpio is installed via apt (python3-lgpio), not pip
```

## Project-Specific Conventions

### Naming Conventions
- **Vision variables**: `bubble_count`, `avg_bubble_size`, `froth_stability` (snake_case)
- **Control parameters**: `kp_frother`, `ki_frother`, `setpoint_bubble_count`
- **Hardware pins**: `FROTHER_PUMP_PIN`, `AGITATOR_PIN`, `AIR_PUMP_PIN`, `FEED_PUMP_PIN`, `ESTOP_PIN` (UPPER_CASE constants)
- **GPIO Pin Assignments**: GPIO 12=Frother, 13=Agitator, 14=Air, 15=Feed, 22=E-Stop
- **Config keys**: Match Python variable names for consistency (`frother_pump`, `agitator`, `air_pump`, `feed_pump`)
- **WebSocket messages**: `{"type": "metrics|frame|anomaly|control", "data": {...}}`

### Code Organization
- **Vision module**: Pure OpenCV processing, no GPIO or control logic
- **Control module**: Stateless PI calculations + pump driver (hardware abstraction)
- **API layer**: Never directly access hardware - always through control/vision modules
- **Config files**: JSON for all tunable parameters (camera settings, PI gains, thresholds)
- **Threading model**: Separate threads for vision capture, processing, control loop, API server
- **Data flow**: Vision → Queue → Control → Pump (unidirectional, avoid circular dependencies)

### Safety & Critical Systems
- **E-STOP**: Hardware-wired (GPIO input + interrupt), software cannot override
- **Watchdog timer**: If vision fails for >5s, halt pump and alert operator
- **Pump limits**: Hard-code max duty cycle at 80% (prevent overdosing)
- **Camera validation**: Check frame rate and quality on startup, fail if < 5 FPS
- **Graceful degradation**: If anomaly detector unavailable, continue with PI control only
- **Data validation**: Reject bubble counts < 0 or > 1000 (sensor error)
- **Configuration bounds**: Validate all config values on load (e.g., Kp > 0, 0 < Ki < 1)

## Important Notes for AI Assistants

### Code Quality & Maintainability Principles
- **Student-Friendly Code**: Write clear, well-commented code that students can easily debug and understand
  - Use descriptive variable names (e.g., `bubble_count` not `bc`)
  - Add inline comments explaining WHY, not just WHAT
  - Break complex operations into smaller, named functions
  - Include example usage in docstrings
  - Maintain high engineering functionality while keeping readability first
  
- **Code Brevity**: Keep code concise without sacrificing clarity
  - Avoid lengthy programs - prefer modular, focused functions (<50 lines)
  - Use Python built-ins and comprehensions where they improve readability
  - Eliminate redundant code through helper functions
  - Single responsibility principle - one function, one job
  - If a file exceeds 200 lines, consider splitting into modules

### Domain-Specific Considerations
- **Froth Characteristics**: More/larger bubbles = over-frothing (reduce dosage), fewer/smaller = under-frothing
- **Control Loop Timing**: Target 1-2 second cycle time (froth responds slowly, faster sampling wastes CPU)
- **Vision vs Reality**: Bubble count is proxy for frother concentration (indirect measurement)
- **Additional Control**: Manual control for agitator, air pump and feed pump in control panel
- **RPi Constraints**: 4GB RAM shared with GPU, avoid >500MB data structures, use generators not lists
- **Reagent Economy**: Overcontrol wastes expensive frother - prefer slight underdamping

### When Writing Code
- **Logging levels**: DEBUG=vision details, INFO=control actions, WARNING=anomalies, ERROR=hardware fails
- **Type hints**: Always use (helps students and debugging on headless RPi)
  ```python
  def analyze_froth(frame: np.ndarray) -> Dict[str, float]:
      """Analyze froth characteristics from camera frame.
      
      Args:
          frame: BGR image from camera (shape: height x width x 3)
      
      Returns:
          Dictionary with bubble_count, avg_size, stability (0-1)
      
      Example:
          >>> metrics = analyze_froth(camera_frame)
          >>> print(f"Bubbles detected: {metrics['bubble_count']}")
      """
  ```
- **Config-driven**: Read thresholds from JSON, never hardcode magic numbers
- **Stateless functions**: Vision/control functions should be pure (testable without hardware)
- **Error propagation**: Raise exceptions for hardware errors, return None for missing data
- **Async API**: Use FastAPI async endpoints to avoid blocking event loop
- **Docstrings**: Include units, valid ranges, sample values, AND examples
  ```python
  def set_pump_rate(duty_cycle: float):
      """Set peristaltic pump PWM duty cycle.
      
      Args:
          duty_cycle: 0-100 (percentage), maps to 0-100 mL/min frother flow
      
      Example:
          >>> set_pump_rate(50)  # 50% duty cycle = 50 mL/min
      """
  ```

### Integration Points

**Vision → Control Pipeline**:
```python
# Queue-based communication (thread-safe)
metrics_queue.put({
    'bubble_count': 150,
    'avg_size': 245.3,
    'timestamp': time.time()
})
```

**FastAPI Endpoints**:
- `GET /api/metrics` - Latest froth metrics (JSON)
- `GET /api/status` - System health (pump, camera, anomaly status)
- `POST /api/setpoint` - Update control setpoint
- `POST /api/override` - Manual pump control (requires confirmation)
- `WS /ws` - Real-time video + metrics stream

**Database Schema** (SQLite):
```sql
CREATE TABLE metrics (
    id INTEGER PRIMARY KEY,
    timestamp DATETIME DEFAULT CURRENT_TIMESTAMP,
    bubble_count INTEGER,
    avg_bubble_size REAL,
    size_std_dev REAL,
    pump_duty_cycle REAL,
    anomaly_detected BOOLEAN
);

CREATE INDEX idx_timestamp ON metrics(timestamp);
```

**Config File Format** (config/control_config.json):
```json
{
    "pi_controller": {
        "kp": 0.5,
        "ki": 0.05,
        "setpoint": 120,
        "output_limits": [0, 80]
    },
    "frother_pump": {
        "pin": 12,
        "frequency_hz": 1000,
        "calibration_ml_per_pwm": 0.1,
        "max_duty_cycle": 80
    },
    "agitator": {
        "pin": 13,
        "frequency_hz": 1000,
        "max_rpm": 1500
    },
    "air_pump": {
        "pin": 14,
        "frequency_hz": 1000
    },
    "feed_pump": {
        "pin": 15,
        "frequency_hz": 1000
    }
}
```

## Development Workflow

### Initial Setup on RPi
```powershell
# On development machine (Windows)
scp -r flotation_project/ pi@raspberrypi.local:/home/pi/
ssh pi@raspberrypi.local
cd flotation_project
bash setup.sh
```

### Testing Without Hardware
```python
# scripts/simulate_froth.py - Generate synthetic bubble images
python scripts/simulate_froth.py --output test_frames/
python src/vision/bubble_detector.py --input test_frames/
```

### Calibration Workflow
1. **Camera**: `python scripts/calibrate_camera.py` (set exposure, focus, white balance)
2. **Pump**: Measure mL dispensed at various PWM values → update config
3. **PI Tuning**: Run `scripts/tune_pi.py` with step input, observe response
4. **Anomaly Training**: Collect 1-2 hours of normal operation → `ml/train_anomaly.py`

### Monitoring & Debugging
- **Dashboard**: http://raspberrypi.local:8000
- **Logs**: `tail -f /var/log/flotation.log`
- **System stats**: `htop`, `vcgencmd measure_temp`
- **Database query**: `sqlite3 flotation.db "SELECT * FROM metrics ORDER BY timestamp DESC LIMIT 10"`

### Performance Benchmarks (Target on RPi 5)
- Vision processing: <100ms per frame (640x480)
- Control loop: <50ms per iteration
- API response: <20ms for metrics endpoint
- WebSocket latency: <200ms frame delivery
- Memory usage: <800MB total
- CPU utilization: <60% average

## Critical Files to Review First
1. `config/control_config.json` - Start here to understand tunable parameters
2. `src/vision/bubble_detector.py` - Core algorithm for froth analysis
3. `src/control/pi_controller.py` - Control loop implementation
4. `src/api/main.py` - System initialization and startup sequence
5. `dashboard/js/app.js` - Frontend logic and WebSocket handling

---

**Note**: This file captures the "why" and "how" that isn't obvious from reading individual source files. Update when architectural decisions change or new patterns emerge.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Abimalek1200) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-09 -->
