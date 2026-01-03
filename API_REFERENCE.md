# LED Tower API Reference

## Quick Reference

### MQTT Topics
- **Command Topic**: `device/cmd` - Send commands to control LEDs
- **State Topic**: `device/state` - Monitor current LED states

### Command Format
```
<color> <action>
```

## Commands

### Individual LED Control

#### Turn LED On
```
red on
yellow on
green on
blue on
```
**Effect**: LED turns on and stays on until changed

#### Turn LED Off
```
red off
yellow off
green off
blue off
```
**Effect**: LED turns off and stays off until changed

#### Fast Flash
```
red fflash
yellow fflash
green fflash
blue fflash
```
**Effect**: LED flashes rapidly (100ms on/off) for 10 cycles, then turns off
**Duration**: ~2 seconds

#### Slow Flash
```
red sflash
yellow sflash
green sflash
blue sflash
```
**Effect**: LED flashes slowly (500ms on/off) for 5 cycles, then turns off
**Duration**: ~5 seconds

### Group LED Control

#### Turn All LEDs On
```
all on
```
**Effect**: All LEDs turn on simultaneously

#### Turn All LEDs Off
```
all off
```
**Effect**: All LEDs turn off simultaneously

## State Monitoring

### State Topic Format
The device publishes its current state to `device/state` as JSON:

```json
{
  "red": "on",
  "yellow": "off", 
  "green": "fflash",
  "blue": "sflash"
}
```

### Possible State Values
- `"on"` - LED is solid on
- `"off"` - LED is off
- `"fflash"` - LED is currently fast flashing
- `"sflash"` - LED is currently slow flashing

## Usage Examples

### MQTT Command Line Examples

Using mosquitto_pub:
```bash
# Turn red LED on
mosquitto_pub -h mqtt_server -t device/cmd -m "red on"

# Fast flash green LED
mosquitto_pub -h mqtt_server -t device/cmd -m "green fflash"

# Turn all LEDs off
mosquitto_pub -h mqtt_server -t device/cmd -m "all off"
```

Using mosquitto_sub to monitor state:
```bash
# Monitor LED states
mosquitto_sub -h mqtt_server -t device/state
```

### Node-RED Example
```json
[
  {
    "id": "led_command",
    "type": "mqtt out",
    "topic": "device/cmd",
    "payload": "red on",
    "broker": "mqtt_broker_config"
  },
  {
    "id": "led_state",
    "type": "mqtt in", 
    "topic": "device/state",
    "broker": "mqtt_broker_config"
  }
]
```

### Python Example
```python
import paho.mqtt.client as mqtt

def on_connect(client, userdata, flags, rc):
    print(f"Connected with result code {rc}")
    client.subscribe("device/state")

def on_message(client, userdata, msg):
    print(f"LED State: {msg.payload.decode()}")

# Create MQTT client
client = mqtt.Client()
client.on_connect = on_connect
client.on_message = on_message

# Connect to broker
client.connect("mqtt_server", 1883, 60)

# Send commands
client.publish("device/cmd", "red on")
client.publish("device/cmd", "green fflash")
client.publish("device/cmd", "all off")

# Start monitoring
client.loop_forever()
```

## Status Patterns Guide

### Typical Status Indications

#### System States
- **Normal Operation**: `green on`
- **Warning**: `yellow on` or `yellow sflash`
- **Error**: `red on` or `red fflash`
- **Processing**: `blue sflash`
- **Off/Idle**: `all off`

#### Build/CI Status
- **Build Running**: `blue sflash`
- **Build Passed**: `green on`
- **Build Failed**: `red on`
- **Build Warning**: `yellow on`

#### Network/Connection Status
- **Connected**: `green on`
- **Connecting**: `blue fflash`
- **Disconnected**: `red on`
- **Timeout**: `yellow fflash`

## Error Handling

### Invalid Commands
- Invalid color names are ignored
- Invalid actions are ignored  
- Malformed commands are ignored
- Device continues normal operation

### Connection Issues
- WiFi disconnection: Device attempts automatic reconnection
- MQTT disconnection: Device attempts reconnection every 5 seconds
- LED states are preserved during brief disconnections

## Hardware Limitations

### Flash Command Behavior
- Flash commands are **blocking operations**
- Device cannot process new commands during flash execution
- LED is unavailable until flash sequence completes
- Consider this when designing automation sequences

### Power Considerations
- All LEDs can be on simultaneously
- Total current draw depends on LED specifications
- Ensure adequate power supply capacity

## Future Enhancements

### Planned Features
- **Continuous Patterns**: Non-blocking flash that continues until changed
- **Fade Effects**: Smooth fade in/out using PWM
- **Brightness Control**: Variable intensity levels
- **Pattern Sequences**: Complex multi-step patterns
- **Timing Parameters**: Configurable flash rates and durations

### Configuration Options
- **Command Persistence**: Save states across reboots
- **Authentication**: Secure MQTT command authentication  
- **Custom Patterns**: User-definable flash sequences
- **Multiple Towers**: Support for multiple LED tower units