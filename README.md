# 🤖 Huenit SDK - Complete Documentation

## 📋 Table of Contents
1. [System Architecture](#system-architecture)
2. [Main Components](#main-components)
3. [Command API](#command-api)
4. [Python Control](#python-control)
5. [Usage Examples](#usage-examples)
6. [Troubleshooting](#troubleshooting)

---

## 🏗️ System Architecture

### Overview
The Huenit SDK is designed as a communication system between a Python application and the Huenit robotic arm hardware. It uses an event-based architecture and serial communication.

```
┌─────────────────┐    ┌──────────────────┐    ┌─────────────────┐
│   Python        │◄──►│   Communicator   │◄──►│   Hardware      │
│   Application   │    │   (Events)       │    │   (Serial)      │
└─────────────────┘    └──────────────────┘    └─────────────────┘
                              │
                              ▼
                       ┌──────────────────┐
                       │  Machine Layer   │
                       │  (Protocol)      │
                       └──────────────────┘
```

### Key Components

#### 1. **Communication System**
- **`communicator.py`**: Event distribution center
- **`machine.py`**: Interface between events and protocol
- **`serial_manager.py`**: Serial connection management

#### 2. **Communication Protocol**
- **`python_protocol_cores/`**: Protocol core
- **`buttons.py`**: Specific command implementations
- **`python_protocol.py`**: Base protocol definitions

#### 3. **Hardware Abstraction**
- **`robot_blocks/robot.py`**: High-level API for arm control
- **`huenit_util.py`**: System utilities
- **`serial_list.py`**: Serial port management

---

## 🔧 Main Components

### 1. Communicator (`communicator.py`)

**Function**: Event distribution center of the system.

**Workflow**:
```python
def catch_event(json_string):
    # 1. Validate JSON
    data = get_valid_json(json_string)
    
    # 2. Verify valid event
    if data == False:
        invalid_event_message(json_string)
        return
    
    # 3. Route to specific function
    function_name = data.get('name')
    if function_name == "playCameraCode":
        # Specific logic
    else:
        machine.catch_event(data)
```

**Supported Events**:
- `Auto_Connect`: Automatic connection
- `Serial_Port_List`: List ports
- `Console_Send`: Send commands
- `Move_Now`: Immediate movement

### 2. Machine Layer (`machine.py`)

**Function**: Interface between events and internal protocol.

```python
def catch_event(data: Dict) -> Dict:
    # 1. Extract event information
    button_name, button_arg = parse_dict_from_app(data)
    
    # 2. Create command object
    obj = parseJsArg(button_name, button_arg, python_path, script_path, connected_info)
    
    # 3. Execute action
    res = obj.mappingAction(obj)
    
    return res
```

### 3. Protocol Core (`python_protocol_cores/`)

#### `main.py` - Entry Point
```python
def main(data: Dict) -> Dict:
    # Configure global variables
    script_path = data.get("script_path")
    python_path = data.get("python_path")
    connected_info = data.get("connected")
    
    # Process command
    button_name, button_arg = parse_dict_from_app(data)
    obj = parseJsArg(button_name, button_arg, python_path, script_path, connected_info)
    res = obj.mappingAction(obj)
    
    return res
```

#### `buttons.py` - Command Implementation
Contains all classes that implement specific commands:
- `BaseConnect`: Base connection
- `SerialPortList`: Port listing
- `AutoConnect`: Automatic connection
- `ConsoleSend`: Command sending
- And many more...

### 4. Robot Control (`robot_blocks/robot.py`)

**High-Level API for Arm Control**:

```python
# Movement
moveG0(x, y, z)           # Fast movement
moveG1(x, y, z)           # Controlled movement
moveAngle(j1, j2, j3, j4, j5, j6)  # Movement by angles
moveZ0()                  # Go to Z=0

# Module Control
suctionOn()               # Activate suction
suctionOff()              # Deactivate suction
gripper(angle)            # Gripper control
pump(state)               # Pump control

# Status
getLoc()                  # Get position
getDeg()                  # Get angles
checkConnection()         # Check connection
```

---

## 📡 Command API

### Event Structure

All events follow this JSON structure:
```json
{
  "name": "EventName",
  "sender": "app",
  "python_path": "/path/to/python",
  "script_path": "/path/to/scripts",
  "connected": {
    "main_arm": null,
    "main_cam": null
  },
  "param": {
    "key": "value"
  }
}
```

### Main Events

#### 1. **Auto_Connect**
Automatically connects to the Huenit device.

```json
{
  "name": "Auto_Connect",
  "sender": "app",
  "python_path": "C:/Program Files/Huenit robotics/resources/app/resources/huenit_py/huenit_env_win/python.exe",
  "script_path": "C:/Program Files/Huenit robotics/resources/app/resources/huenit_py/src",
  "connected": {
    "main_arm": null,
    "main_cam": null
  }
}
```

#### 2. **Serial_Port_List**
Lists all available serial ports.

```json
{
  "name": "Serial_Port_List",
  "sender": "app",
  "python_path": "C:/Program Files/Huenit robotics/resources/app/resources/huenit_py/huenit_env_win/python.exe",
  "script_path": "C:/Program Files/Huenit robotics/resources/app/resources/huenit_py/src"
}
```

#### 3. **Console_Send**
Sends direct commands to hardware.

```json
{
  "name": "Console_Send",
  "sender": "app",
  "python_path": "C:/Program Files/Huenit robotics/resources/app/resources/huenit_py/huenit_env_win/python.exe",
  "script_path": "C:/Program Files/Huenit robotics/resources/app/resources/huenit_py/src",
  "param": {
    "hw_command": "G0 X10 Y20 Z5"
  }
}
```

#### 4. **Move_Now**
Executes immediate movement.

```json
{
  "name": "Move_Now",
  "sender": "app",
  "python_path": "C:/Program Files/Huenit robotics/resources/app/resources/huenit_py/huenit_env_win/python.exe",
  "script_path": "C:/Program Files/Huenit robotics/resources/app/resources/huenit_py/src",
  "param": {
    "x": 10.0,
    "y": 20.0,
    "z": 5.0
  }
}
```

### Supported G-code/M-code Commands

#### Movement
- `G0 X# Y# Z#`: Fast movement
- `G1 X# Y# Z#`: Controlled movement
- `M114`: Get current position
- `M1008 A5`: Go to HOME

#### Module Control
- `M1111`: Activate suction
- `M1112`: Deactivate suction
- `M1113 A#`: Gripper control (angle)
- `M1114 C`: Pump control

#### System Status
- `M122 B`: System information
- `M400`: Wait for movement to finish

---

## 🐍 Python Control

### Environment Setup

#### 1. **Directory Structure**
```
huenit_py/
├── huenit_env_win/          # Python environment for Windows
│   ├── python.exe
│   └── Lib/site-packages/
├── huenit_env_mac/          # Python environment for macOS
│   ├── bin/python3.10
│   └── lib/python3.10/site-packages/
└── src/                     # Source code
    ├── communicator.py
    ├── machine.py
    ├── huenit_util.py
    ├── global_event_name.py
    ├── test.py
    └── python_protocol_cores/
```

#### 2. **Path Configuration**
```python
import sys
import os

# Add Huenit environment to path
if sys.platform == "win32":
    huenit_env = os.path.join(parent_dir, "huenit_env_win", "Lib", "site-packages")
else:
    huenit_env = os.path.join(parent_dir, "huenit_env_mac", "lib", "python3.10", "site-packages")

sys.path.insert(0, huenit_env)
```

### Control Methods

#### 1. **Direct Method (Recommended)**
Use Huenit's event system:

```python
import sys
import os
import json
import subprocess

class HuenitController:
    def __init__(self):
        self.python_executable = self._get_huenit_python()
        self.script_path = os.path.dirname(os.path.abspath(__file__))
        
    def _get_huenit_python(self):
        if sys.platform == "win32":
            return "C:/Program Files/Huenit robotics/resources/app/resources/huenit_py/huenit_env_win/python.exe"
        else:
            return "/path/to/huenit_env_mac/bin/python3.10"
    
    def send_event(self, event_name, params=None):
        """Send event to Huenit system"""
        data = {
            "name": event_name,
            "sender": "app",
            "python_path": self.python_executable,
            "script_path": self.script_path,
            "connected": {"main_arm": None, "main_cam": None}
        }
        
        if params:
            data["param"] = params
            
        json_data = json.dumps(data)
        
        cmd = [self.python_executable, "test.py", json_data]
        result = subprocess.run(cmd, capture_output=True, text=True, cwd=self.script_path)
        
        return result.returncode == 0
    
    def connect(self):
        """Connect to Huenit arm"""
        return self.send_event("Auto_Connect")
    
    def move_to(self, x, y, z):
        """Move arm to specific position"""
        return self.send_event("Console_Send", {"hw_command": f"G0 X{x} Y{y} Z{z}"})
    
    def home(self):
        """Go to home position"""
        return self.send_event("Console_Send", {"hw_command": "M1008 A5"})

# Usage
controller = HuenitController()
controller.connect()
controller.move_to(10, 20, 5)
controller.home()
```

#### 2. **High-Level API Method**
Use functions from `robot_blocks/robot.py`:

```python
import sys
import os
sys.path.append("src")

from python_protocol_cores.robot_blocks.robot import *

# Setup connection
def setup_connection():
    # System configures automatically
    pass

# Arm control
def control_arm():
    # Movements
    moveG0(10, 20, 5)      # Fast movement
    moveG1(15, 25, 8)      # Controlled movement
    moveAngle(0, 45, 90, 0, 0, 0)  # By angles
    
    # Module control
    suctionOn()            # Activate suction
    time.sleep(2)
    suctionOff()           # Deactivate suction
    
    # Status
    pos = getLoc()         # Current position
    angles = getDeg()      # Current angles
    print(f"Position: {pos}")
    print(f"Angles: {angles}")

# Execute
setup_connection()
control_arm()
```

#### 3. **Direct Serial Communication Method**
For low-level control:

```python
import serial
import time

class DirectHuenitControl:
    def __init__(self, port):
        self.ser = serial.Serial(port, 115200, timeout=5)
        time.sleep(2)  # Wait for initialization
        
    def send_command(self, command):
        """Send direct command to arm"""
        if not command.endswith('\n'):
            command += '\n'
            
        self.ser.write(command.encode())
        response = self.ser.readline().decode().strip()
        return response
    
    def move_to(self, x, y, z):
        """Move to specific position"""
        command = f"G0 X{x} Y{y} Z{z}"
        response = self.send_command(command)
        self.send_command("M400")  # Wait for movement
        return response
    
    def get_position(self):
        """Get current position"""
        return self.send_command("M114")
    
    def home(self):
        """Go to home"""
        return self.send_command("M1008 A5")
    
    def close(self):
        """Close connection"""
        self.ser.close()

# Usage
controller = DirectHuenitControl("COM7")  # Port on Windows
controller.move_to(10, 20, 5)
pos = controller.get_position()
controller.home()
controller.close()
```

---

## 💡 Usage Examples

### 1. **Basic Movement Control**
```python
#!/usr/bin/env python3
"""
Basic Huenit arm control example
"""

import sys
import os
import json
import subprocess

def send_huenit_command(event_name, params=None):
    """Send command to Huenit system"""
    data = {
        "name": event_name,
        "sender": "app",
        "python_path": "C:/Program Files/Huenit robotics/resources/app/resources/huenit_py/huenit_env_win/python.exe",
        "script_path": "C:/Program Files/Huenit robotics/resources/app/resources/huenit_py/src",
        "connected": {"main_arm": None, "main_cam": None}
    }
    
    if params:
        data["param"] = params
    
    json_data = json.dumps(data)
    
    cmd = [
        "C:/Program Files/Huenit robotics/resources/app/resources/huenit_py/huenit_env_win/python.exe",
        "test.py",
        json_data
    ]
    
    result = subprocess.run(cmd, capture_output=True, text=True, 
                          cwd="C:/Program Files/Huenit robotics/resources/app/resources/huenit_py/src")
    
    return result.returncode == 0

def main():
    print("🤖 Basic Huenit Arm Control")
    print("=" * 40)
    
    # 1. Connect
    print("🔌 Connecting...")
    if send_huenit_command("Auto_Connect"):
        print("✅ Connected successfully")
    else:
        print("❌ Connection error")
        return
    
    # 2. Go home
    print("🏠 Going to HOME...")
    send_huenit_command("Console_Send", {"hw_command": "M1008 A5"})
    
    # 3. Test movement
    positions = [
        (10, 10, 10),
        (20, 20, 20),
        (30, 30, 30),
        (0, 0, 0)
    ]
    
    for x, y, z in positions:
        print(f"📍 Moving to X:{x} Y:{y} Z:{z}")
        send_huenit_command("Console_Send", {"hw_command": f"G0 X{x} Y{y} Z{z}"})
        send_huenit_command("Console_Send", {"hw_command": "M400"})  # Wait
    
    print("✅ Sequence completed")

if __name__ == "__main__":
    main()
```

### 2. **Keyboard Control Interface**
```python
#!/usr/bin/env python3
"""
Huenit arm control with keyboard
"""

import sys
import os
import json
import subprocess
import keyboard
import time

class HuenitKeyboardControl:
    def __init__(self):
        self.python_executable = "C:/Program Files/Huenit robotics/resources/app/resources/huenit_py/huenit_env_win/python.exe"
        self.script_path = "C:/Program Files/Huenit robotics/resources/app/resources/huenit_py/src"
        self.step_size = 10.0
        self.current_pos = [0, 0, 0]
        
    def send_command(self, event_name, params=None):
        """Send command to Huenit system"""
        data = {
            "name": event_name,
            "sender": "app",
            "python_path": self.python_executable,
            "script_path": self.script_path,
            "connected": {"main_arm": None, "main_cam": None}
        }
        
        if params:
            data["param"] = params
        
        json_data = json.dumps(data)
        
        cmd = [self.python_executable, "test.py", json_data]
        result = subprocess.run(cmd, capture_output=True, text=True, cwd=self.script_path)
        
        return result.returncode == 0
    
    def connect(self):
        """Connect to arm"""
        print("🔌 Connecting to Huenit arm...")
        return self.send_command("Auto_Connect")
    
    def move_relative(self, dx, dy, dz):
        """Move relatively"""
        new_x = self.current_pos[0] + dx
        new_y = self.current_pos[1] + dy
        new_z = self.current_pos[2] + dz
        
        if self.send_command("Console_Send", {"hw_command": f"G0 X{new_x} Y{new_y} Z{new_z}"}):
            self.current_pos = [new_x, new_y, new_z]
            print(f"📍 Position: X:{new_x} Y:{new_y} Z:{new_z}")
    
    def go_home(self):
        """Go to home"""
        if self.send_command("Console_Send", {"hw_command": "M1008 A5"}):
            self.current_pos = [0, 0, 0]
            print("🏠 At HOME")
    
    def setup_keyboard_controls(self):
        """Setup keyboard controls"""
        keyboard.on_press_key('up', lambda _: self.move_relative(0, self.step_size, 0))
        keyboard.on_press_key('down', lambda _: self.move_relative(0, -self.step_size, 0))
        keyboard.on_press_key('left', lambda _: self.move_relative(-self.step_size, 0, 0))
        keyboard.on_press_key('right', lambda _: self.move_relative(self.step_size, 0, 0))
        keyboard.on_press_key('w', lambda _: self.move_relative(0, 0, self.step_size))
        keyboard.on_press_key('s', lambda _: self.move_relative(0, 0, -self.step_size))
        keyboard.on_press_key('h', lambda _: self.go_home())
        keyboard.on_press_key('+', lambda _: self.change_step(5))
        keyboard.on_press_key('-', lambda _: self.change_step(-5))
        keyboard.on_press_key('q', lambda _: self.quit())
    
    def change_step(self, delta):
        """Change step size"""
        self.step_size = max(1.0, min(50.0, self.step_size + delta))
        print(f"📏 Step: {self.step_size:.1f}mm")
    
    def quit(self):
        """Exit program"""
        print("\n👋 Exiting...")
        sys.exit(0)
    
    def run(self):
        """Run main program"""
        print("🤖 Huenit Arm Keyboard Control")
        print("=" * 50)
        
        if not self.connect():
            print("❌ Could not connect")
            return
        
        print("\n🎮 CONTROLS:")
        print("  ↑ ↓ ← → : Move in X,Y")
        print("  W S      : Move in Z")
        print("  H        : Go to HOME")
        print("  + -      : Change step")
        print("  Q        : Exit")
        print(f"\n📏 Current step: {self.step_size:.1f}mm")
        
        self.setup_keyboard_controls()
        
        print("\n✅ Ready. Use keys to control the arm.")
        
        try:
            while True:
                time.sleep(0.1)
        except KeyboardInterrupt:
            self.quit()

if __name__ == "__main__":
    controller = HuenitKeyboardControl()
    controller.run()
```

---

## 🐛 Troubleshooting

### Common Problems

#### 1. **Error: "No Huenit device found"**
**Problem**: Device appears as "USB Serial Port (COM7)" instead of "HUENIT"

**Solution**:
```python
# In huenit_util.py, modify detection function:
def is_huenit_usb(port):
    # Search for both "HUENIT" and "USB Serial Port"
    description = port.description.upper()
    return 'HUENIT' in description or 'USB SERIAL PORT' in description
```

#### 2. **Error: "IndexError: list index out of range"**
**Problem**: System expects arguments that are not being passed

**Solution**: Use complete JSON with all paths:
```json
{
  "name": "Auto_Connect",
  "sender": "app",
  "python_path": "C:/Program Files/Huenit robotics/resources/app/resources/huenit_py/huenit_env_win/python.exe",
  "script_path": "C:/Program Files/Huenit robotics/resources/app/resources/huenit_py/src",
  "connected": {"main_arm": null, "main_cam": null}
}
```

#### 3. **Error: "UnicodeDecodeError"**
**Problem**: Encoding issues on Windows

**Solution**: Specify UTF-8 encoding:
```python
exec(open('test.py', encoding='utf-8').read())
```

#### 4. **Error: "module 'SerialPortList' has no attribute 'SerialPortList'"
**Problem**: System cannot find the correct function in buttons.py

**Solution**: Verify that the event name exactly matches the function in buttons.py

### Debugging

#### 1. **Verify Serial Connection**
```python
import serial.tools.list_ports

ports = list_ports.comports()
for port in ports:
    print(f"Port: {port.device} - {port.description}")
```

#### 2. **Test Individual Commands**
```python
# Test simple command
controller.send_command("Console_Send", {"hw_command": "M114"})

# Verify response
pos = controller.get_position()
print(f"Position: {pos}")
```

#### 3. **System Logs**
The Huenit system generates detailed logs. Look for:
- `fake_log`: Registered events
- `command =`: Sent commands
- `msg from hw =`: Hardware responses

---

## 📚 References

### Key SDK Files
- `src/communicator.py`: Event system
- `src/machine.py`: Protocol interface
- `src/huenit_util.py`: System utilities
- `src/global_event_name.py`: Event definitions
- `src/python_protocol_cores/main.py`: Entry point
- `src/python_protocol_cores/buttons.py`: Command implementation
- `src/python_protocol_cores/robot_blocks/robot.py`: High-level API

### Useful G-code/M-code Commands
- `G0 X# Y# Z#`: Fast movement
- `G1 X# Y# Z#`: Controlled movement
- `M114`: Current position
- `M1008 A5`: Home
- `M1111`: Suction ON
- `M1112`: Suction OFF
- `M400`: Wait for movement

### Response Structure
```json
{
  "name": "EventName",
  "success": true,
  "sender": "python",
  "param": {
    "result": "data"
  }
}
```

---

**© 2024 - Huenit SDK Documentation**
