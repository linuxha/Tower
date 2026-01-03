# LED Tower Troubleshooting Guide

## Quick Diagnostics Checklist

1. **Power**: Is the LED tower receiving 24VDC power?
2. **WiFi**: Is the ESP32 connected to your network?
3. **MQTT**: Can the device connect to the MQTT broker?
4. **Hardware**: Are all LEDs functional when manually controlled?
5. **Commands**: Are MQTT commands properly formatted?

## Common Issues and Solutions

### LEDs Not Lighting

#### Symptom: No LEDs turn on with any command

**Possible Causes:**
- Power supply issues
- Incorrect wiring
- MOSFET configuration problems
- GPIO pin assignments

**Solutions:**
1. **Check Power Supply**
   - Verify 24VDC is present at LED tower input
   - Check manual switch is in ON position
   - Measure voltage at controller board input

2. **Verify Wiring**
   ```
   ESP32 Pin → MOSFET Gate → LED+
   GPIO 4    → Red Gate    → Red LED
   GPIO 5    → Yellow Gate → Yellow LED  
   GPIO 6    → Green Gate  → Green LED
   GPIO 7    → Blue Gate   → Blue LED
   ```

3. **Test MOSFET Operation**
   - Measure gate voltage when GPIO is HIGH (should be ~3.3V)
   - Check MOSFET drain-source continuity when gate is active
   - Verify N-Channel MOSFETs are used (positive gate activation)

4. **Manual GPIO Test**
   Add to setup() for direct testing:
   ```cpp
   // Test all LEDs on startup
   digitalWrite(redPin, HIGH);
   delay(1000);
   digitalWrite(yellowPin, HIGH); 
   delay(1000);
   digitalWrite(greenPin, HIGH);
   delay(1000);
   digitalWrite(bluePin, HIGH);
   delay(1000);
   ```

#### Symptom: Some LEDs work, others don't

**Likely Causes:**
- Individual LED failure
- Specific MOSFET failure
- Wiring issue on non-working channels

**Solutions:**
1. **Test Individual LEDs**
   - Temporarily connect working LED to non-working channel
   - If LED works, problem is in that channel's circuit
   - If LED doesn't work, problem is the LED itself

2. **Check GPIO Pin Assignment**
   - Verify correct pin numbers in code match physical wiring
   - Try swapping pin assignments in software to isolate hardware issues

### Network Connectivity Issues

#### Symptom: Device not connecting to WiFi

**Debugging Steps:**
1. **Check Credentials**
   - Verify SSID and password in [myWiFi.h](myWiFi.h)
   - Ensure no extra spaces or special characters
   - Check for case sensitivity

2. **Add Serial Debug Output**
   ```cpp
   void setup_wifi() {
     Serial.begin(115200);
     delay(10);
     Serial.print("Connecting to ");
     Serial.println(ssid);
     
     WiFi.begin(ssid, password);
     
     while (WiFi.status() != WL_CONNECTED) {
       delay(500);
       Serial.print(".");
     }
     
     Serial.println("");
     Serial.println("WiFi connected");
     Serial.print("IP address: ");
     Serial.println(WiFi.localIP());
   }
   ```

3. **Check Network Requirements**
   - Ensure 2.4GHz network (ESP32 doesn't support 5GHz)
   - Check for MAC address filtering
   - Verify network allows new device connections

#### Symptom: WiFi connects but MQTT fails

**Debugging Steps:**
1. **Verify MQTT Broker Settings**
   - Check server address in [myMQTT.h](myMQTT.h)
   - Verify port 1883 is accessible
   - Test broker connectivity from another client

2. **Add MQTT Connection Debug**
   ```cpp
   void reconnect() {
     while (!client.connected()) {
       Serial.print("Attempting MQTT connection...");
       if (client.connect("ESP32Client")) {
         Serial.println("connected");
         client.subscribe(command_topic);
       } else {
         Serial.print("failed, rc=");
         Serial.print(client.state());
         Serial.println(" retry in 5 seconds");
         delay(5000);
       }
     }
   }
   ```

3. **MQTT Connection Error Codes**
   - `-4`: Connection timeout
   - `-3`: Connection lost  
   - `-2`: Connect failed
   - `-1`: Disconnected
   - `1`: Bad protocol version
   - `2`: Client ID rejected
   - `3`: Server unavailable
   - `4`: Bad username/password
   - `5`: Not authorized

### Command Processing Issues

#### Symptom: Commands sent but LEDs don't respond

**Troubleshooting:**
1. **Verify Topic Subscription**
   - Ensure device subscribes to correct command topic
   - Check topic name matches exactly (case sensitive)

2. **Test Command Format**
   ```bash
   # Correct format
   mosquitto_pub -h mqtt_server -t device/cmd -m "red on"
   
   # Common mistakes
   mosquitto_pub -h mqtt_server -t device/cmd -m "Red On"  # Wrong case
   mosquitto_pub -h mqtt_server -t device/cmd -m "red_on"  # Wrong separator
   ```

3. **Add Command Debug Output**
   ```cpp
   void callback(char* topic, byte* payload, unsigned int length) {
     String message;
     for (int i = 0; i < length; i++) {
       message += (char)payload[i];
     }
     Serial.print("Received command: ");
     Serial.println(message);
     processCommand(message);
   }
   ```

#### Symptom: Flash commands don't work properly

**Common Issues:**
- Flash commands are blocking - no new commands accepted during execution
- Device appears unresponsive during flash sequences

**Solutions:**
1. **Wait for Flash Completion**
   - Fast flash: ~2 seconds
   - Slow flash: ~5 seconds
   - Send next command after completion

2. **Implement Non-Blocking Flash (Future Enhancement)**
   ```cpp
   // Pseudo-code for non-blocking implementation
   unsigned long flashStartTime;
   bool flashActive = false;
   int flashCount = 0;
   
   void loop() {
     if (!client.connected()) {
       reconnect();
     }
     
     // Handle ongoing flash patterns
     if (flashActive) {
       handleFlashPattern();
     }
     
     client.loop();
   }
   ```

### State Monitoring Issues

#### Symptom: State topic not updating

**Debugging:**
1. **Check Topic Publishing**
   ```cpp
   void publishState() {
     String output;
     serializeJson(ledState, output);
     Serial.print("Publishing state: ");
     Serial.println(output);
     client.publish(state_topic, output.c_str());
   }
   ```

2. **Verify JSON Serialization**
   - Ensure ArduinoJson library is properly installed
   - Check JSON buffer size (1024 bytes should be sufficient)

3. **Test State Subscription**
   ```bash
   # Monitor state updates
   mosquitto_sub -h mqtt_server -t device/state -v
   ```

### Hardware Assembly Issues

#### MOSFET Selection and Configuration

**N-Channel MOSFET Requirements:**
- Logic-level gate (3.3V compatible)
- Adequate current rating for LED load
- Low RDS(on) resistance for efficiency

**Correct Wiring:**
```
ESP32 GPIO → MOSFET Gate
+24V → LED Anode
LED Cathode → MOSFET Drain  
MOSFET Source → Ground
```

**Common Wiring Mistakes:**
- Using P-Channel MOSFETs (need negative gate voltage)
- Incorrect drain/source connections
- Missing gate resistors (can cause oscillation)

#### LED Tower Wiring

**Standard Color Codes:**
- Red wire → Red LED
- Yellow wire → Yellow LED
- Green wire → Green LED
- Blue wire → Blue LED  
- Black wire → Common ground
- White ribbon cable → Multi-conductor connection

**Verification Steps:**
1. Use multimeter to verify LED polarity
2. Test each LED individually with 24V supply
3. Confirm continuity from ESP32 pins to MOSFET gates

### Development Environment Issues

#### Arduino IDE Setup

**Required Board Support:**
```
File → Preferences → Additional Board Manager URLs:
https://dl.espressif.com/dl/package_esp32_index.json
```

**Board Selection:**
- Board: "ESP32S2 Dev Module" 
- Upload Speed: "921600"
- CPU Frequency: "240MHz"
- Flash Mode: "QIO"
- Flash Size: "4MB"

**Required Libraries:**
```cpp
#include <WiFi.h>        // Built-in
#include <PubSubClient.h> // Install via Library Manager
#include <ArduinoJson.h> // Install via Library Manager
```

#### Upload Issues

**DFU Mode Entry:**
1. Hold Button 0 (BOOT)
2. Press and release Button Reset
3. Release Button 0 when USB reconnection sound occurs
4. Upload within 10 seconds

**Common Upload Errors:**
- "Failed to connect": Device not in DFU mode
- "Timeout": USB cable or driver issues
- "Permission denied": Linux users may need udev rules

### Performance Optimization

#### Memory Usage
- JSON document size: Currently 1KB (adequate)
- String concatenation: Consider using char arrays for better performance
- Flash memory: ~4MB available, current usage minimal

#### Network Efficiency
- MQTT keepalive: Default 15 seconds
- Reconnection delay: 5 seconds (may be too aggressive for poor networks)
- WiFi power management: Consider enabling for battery applications

## Debugging Tools and Techniques

### Serial Monitor Output
Add comprehensive logging:
```cpp
void setup() {
  Serial.begin(115200);
  Serial.println("LED Tower Starting...");
  
  // Rest of setup code with Serial.println() debugging
}
```

### MQTT Client Testing
```bash
# Test broker connectivity
mosquitto_pub -h mqtt_server -t test -m "hello"

# Monitor all topics
mosquitto_sub -h mqtt_server -t '#' -v

# Test device commands
mosquitto_pub -h mqtt_server -t device/cmd -m "red on"
mosquitto_sub -h mqtt_server -t device/state
```

### Network Analysis
```bash
# Ping test
ping mqtt_server

# Port connectivity test  
telnet mqtt_server 1883

# WiFi signal strength
iwconfig  # Linux
netsh wlan show profiles  # Windows
```

### Hardware Testing
```bash
# GPIO state reading (if SSH access available)
cat /sys/class/gpio/gpio4/value

# Voltage measurements with multimeter
# Gate voltage: ~3.3V when HIGH
# LED voltage: ~24V when on
```

## When to Contact Support

Contact the developer or community when:

1. **Hardware Failures**
   - Multiple MOSFETs failing simultaneously
   - ESP32 board damage or malfunction
   - Power supply instability

2. **Persistent Network Issues** 
   - WiFi connects but MQTT consistently fails
   - Intermittent disconnections without clear cause
   - Performance degradation over time

3. **Code Enhancement Requests**
   - Non-blocking flash patterns needed
   - Integration with specific home automation systems
   - Custom pattern requirements

4. **Security Concerns**
   - MQTT authentication implementation
   - Network security hardening
   - Credential management improvements

## Preventive Maintenance

### Regular Checks
- **Monthly**: Verify all LEDs still function correctly
- **Quarterly**: Check network connectivity stability  
- **Annually**: Update ESP32 firmware and libraries

### Best Practices
- Use version control for configuration changes
- Document any hardware modifications
- Test commands in development environment first
- Monitor MQTT broker logs for anomalies
- Keep spare MOSFETs and ESP32 boards available