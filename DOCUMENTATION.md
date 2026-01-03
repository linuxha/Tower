# LED Tower Documentation

## Project Overview

The LED Tower is an ESP32-based IoT device that controls a Red/Yellow/Green/Blue LED tower for status indication. The system receives commands via MQTT and provides real-time state feedback, making it ideal for monitoring ongoing events, system statuses, or notifications.

## Hardware Specifications

### ESP32 S2 Mini Board
- **Board**: Wemos ESP32 S2 Mini
- **Features**: 
  - ESP32-S2FN4R2 WIFI IC
  - Type-C USB
  - 4MB Flash, 2MB PSRAM
  - 27 GPIO pins
  - Compatible with Arduino IDE

### GPIO Pin Assignments
- **GPIO 4**: Red LED
- **GPIO 5**: Yellow LED  
- **GPIO 6**: Green LED
- **GPIO 7**: Blue LED

### Power Supply
- **Input**: 24VDC
- **Control**: Manual switch → Controller board → LEDs
- **Switching**: N-Channel MOSFETs (Positive Gate activation)

### LED Tower Wiring
```
24VDC → Manual Switch → Controller Board → LEDs
White wire: Red LED
Yellow wire: Yellow LED  
Green wire: Green LED
Blue wire: Blue LED
Black wire: Common ground
```

## Software Architecture

### Dependencies
- `WiFi.h` - ESP32 WiFi connectivity
- `PubSubClient.h` - MQTT client functionality
- `ArduinoJson.h` - JSON state management

### Configuration Files
- **[myWiFi.h](myWiFi.h)**: WiFi credentials (excluded from version control)
- **[myMQTT.h](myMQTT.h)**: MQTT broker settings (excluded from version control)

### Core Functionality

#### WiFi Connection
- Automatic connection on startup
- Reconnection handling in main loop

#### MQTT Integration
- **Broker**: Configurable MQTT server
- **Command Topic**: `device/cmd` - Receives control commands
- **State Topic**: `device/state` - Reports current LED states in JSON format
- **Client ID**: `ESP32Client`

#### LED Control
- Individual LED control (Red, Yellow, Green, Blue)
- Group control (All LEDs simultaneously)
- Real-time state tracking with JSON state object

## Command Interface

### Command Format
Commands follow the pattern: `<color> <action>`

### Supported Colors
- `red` - Controls red LED
- `yellow` - Controls yellow LED  
- `green` - Controls green LED
- `blue` - Controls blue LED
- `all` - Controls all LEDs simultaneously

### Supported Actions
- `on` - Turn LED on (solid)
- `off` - Turn LED off
- `fflash` - Fast flash (100ms on/off, 10 cycles)
- `sflash` - Slow flash (500ms on/off, 5 cycles)

### Command Examples
```
red on          # Turn red LED on
yellow off      # Turn yellow LED off
green fflash    # Fast flash green LED
blue sflash     # Slow flash blue LED
all on          # Turn all LEDs on
all off         # Turn all LEDs off
```

## State Monitoring

The device publishes its current state to the `device/state` topic in JSON format:

```json
{
  "red": "on",
  "yellow": "off", 
  "green": "fflash",
  "blue": "sflash"
}
```

State values can be: `on`, `off`, `fflash`, `sflash`

## Code Structure

### Main Functions

#### `setup()`
- Initializes GPIO pins as outputs
- Establishes WiFi connection
- Configures MQTT client and callback
- Initializes LED state object

#### `loop()`
- Maintains MQTT connection
- Processes incoming messages
- Handles reconnection if needed

#### `callback(char* topic, byte* payload, unsigned int length)`
- Processes incoming MQTT messages
- Converts payload to string and passes to command processor

#### `processCommand(String command)`
- Parses color and action from command string
- Routes to appropriate LED control function
- Updates state and publishes changes

#### `processAllCommand(String action)`
- Handles "all" commands affecting multiple LEDs
- Updates all LED states simultaneously

#### `publishState()`
- Serializes current LED state to JSON
- Publishes state to MQTT state topic

### Utility Functions

#### `setup_wifi()`
- Connects to configured WiFi network
- Blocks until connection established

#### `reconnect()`
- Handles MQTT broker reconnection
- Subscribes to command topic on successful connection

#### `rot47(char *s)`
- ROT47 encoding function (currently unused)
- Available for future security features

## Installation and Setup

### Prerequisites
1. Arduino IDE with ESP32 board support
2. Required libraries:
   - WiFi (built-in)
   - PubSubClient
   - ArduinoJson

### Configuration
1. Update `myWiFi.h` with your WiFi credentials
2. Update `myMQTT.h` with your MQTT broker details
3. Verify GPIO pin assignments match your hardware

### Deployment
1. Connect ESP32 to computer via USB-C
2. Enter DFU mode (hold Button 0, press Reset, release Button 0)
3. Upload code through Arduino IDE

## Current Limitations and Future Enhancements

### Known Issues
- Flash commands are blocking (fixed duration, not continuous)
- No fade functionality implemented yet
- Commands cannot be interrupted once started

### Planned Improvements
- Non-blocking flash patterns that continue until changed
- Fade in/out functionality with PWM control
- Command queuing and interruption
- Brightness control
- Pattern sequences
- Status persistence across reboots

## Security Notes

- WiFi and MQTT credentials are stored in separate header files
- Use `git update-index --assume-unchanged` for credential files
- Consider implementing authentication for MQTT commands
- ROT47 encoding function available for basic obfuscation

## Testing and Troubleshooting

### Initial Testing Checklist
1. Verify all LEDs light up individually with manual control
2. Check WiFi connectivity
3. Confirm MQTT broker connection
4. Test each command via MQTT client
5. Monitor state topic for correct reporting

### Common Issues
- **LEDs not lighting**: Check wiring and MOSFET configuration
- **WiFi connection fails**: Verify credentials in myWiFi.h
- **MQTT not connecting**: Check broker address and port in myMQTT.h
- **Commands not working**: Verify topic subscription and command format

## Hardware Assembly Notes

The project uses N-Channel MOSFETs for LED switching:
- Positive gate voltage turns switch ON
- Ensure proper current handling for LED load
- Consider heat dissipation for high-power LEDs

## Development History

This project was created collaboratively using AI assistants (ChatGPT 4, Bard, Hugging Face) with the developer making refinements and customizations. The code represents a learning exercise in AI-assisted embedded development.

**Project Date**: June 13, 2023
**Last Updated**: January 2, 2026