# LED Tower Control System - Copilot Instructions

## Project Overview
ESP32-based IoT device controlling a 4-color LED tower (Red/Yellow/Green/Blue) via MQTT commands. Uses N-Channel MOSFETs for LED switching and provides real-time status monitoring.

## Architecture & Key Components

### Hardware Configuration
- **ESP32 S2 Mini**: Main controller with WiFi capability
- **GPIO Pins**: 4→Red, 5→Yellow, 6→Green, 7→Blue (N-Channel MOSFET control)
- **Power**: 24VDC external supply for LED tower
- **Connectivity**: WiFi + MQTT over port 1883

### Code Structure
- **Tower.ino**: Main application with setup(), loop(), and command processing
- **myWiFi.h**: WiFi credentials (git-ignored for security)
- **myMQTT.h**: MQTT server configuration (git-ignored)
- **t/**: Template directory with example configuration files

## Development Patterns

### MQTT Command Protocol
```cpp
// Command format: "<color> <action>"
// Colors: red, yellow, green, blue, all
// Actions: on, off, fflash, sflash
processCommand("red on");        // Solid on
processCommand("blue fflash");   // Fast flash (10 cycles, 100ms)
processCommand("all off");       // Turn off all LEDs
```

### State Management
- Global `ledState` JSON object tracks all LED states
- Automatically published to `device/state` topic after each command
- Uses ArduinoJson library for serialization

### Security Configuration
Configuration files use git's `assume-unchanged` flag:
```bash
git update-index --assume-unchanged myWiFi.h
git update-index --assume-unchanged myMQTT.h
```

## Critical Development Considerations

### Blocking vs Non-Blocking Operations
**Current Issue**: Flash commands (`fflash`/`sflash`) are blocking - device cannot receive new commands during flash sequences. This is a known limitation documented in TODO comments.

**When modifying**: Consider implementing non-blocking patterns using timers or state machines for continuous operations.

### MQTT Topics
- **Command Input**: `device/cmd` - receives control commands
- **State Output**: `device/state` - publishes JSON state after each change

### Error Handling
- WiFi auto-reconnection in `setup_wifi()`
- MQTT reconnection with 5-second retry in `reconnect()`
- No error handling for malformed commands (improvement opportunity)

## Testing & Debugging

### Hardware Verification Workflow
1. Test individual LEDs manually before ESP32 integration
2. Verify MOSFET switching with multimeter
3. Check 24V power supply and ground connections
4. Monitor MQTT topics for command/state verification

### Development Environment
- **Platform**: Arduino IDE/PlatformIO with ESP32 Arduino Core
- **Key Libraries**: WiFi.h, PubSubClient.h, ArduinoJson.h
- **Upload Mode**: DFU mode (Hold Button 0 + Reset, then release Button 0)

## Common Modification Patterns

### Adding New Commands
1. Extend `processCommand()` switch statements
2. Update `ledState` JSON object
3. Call `publishState()` after state changes
4. Document new command format in header comments

### Adding New LEDs/Colors
1. Define new GPIO pin constant
2. Add pinMode() in setup()
3. Extend color parsing in `processCommand()`
4. Add to `processAllCommand()` if applicable
5. Update JSON state structure

### Implementing Non-Blocking Patterns
Consider using `millis()` timers instead of `delay()` for flash/fade operations to maintain MQTT responsiveness.