# LEDTower
<img align="left" src="images/Tower.png" alt="4 colored LED Tower">
ESP32 controlling a 24v R/Y/G/Bl LED Tower via MQTT

I wrote this state machine a few years ago as a proof of concept but didn't build the tower. So now  I'm trying to learn more about the use of AI in various projects and Tower is one of those projects to use. I'm no expert with the new AI code tools and boy am I learning a lot! I do know how to write the code to accomplish this program. This is another experiment with Copilot, Gemini, ChatGPT 4, Bard and Huggin Face. I was only accepting the ChatGPT generated code as the others are not correct but I've since switched to using Github Copilot Pro. I'm paying for the Github Copilot Pro subscription (as opposed to the Github Pro subscription) so I can get more proficient with AIs. By paying for the subscription I get more tokens and the AI seems to remember a lot more of earlier chats (less hallucinations). I'll experiment with different prompts and requiments to see if I can get the others working better. The requirements below are part of what I used in the first attempts. Using only requirements doesn't seem to work as well as a text description.
<br clear="left"/>

## VS Code

Still working on this. I can manually get Code to compile the ESP32 code or I can use the latest Arduino IDE. But using the built in methods is still a work in progress. These seem tow rok for me:

```bash
arduino-cli board listall "LOLIN S2 Mini"

# Not sure these are the correct libraries
arduino-cli lib install "WiFi" "MQTT Client"

arduino-cli compile --fqbn esp32:esp32:lolin_s2_mini

# Upload
arduino-cli upload -p /dev/ttyACM0 --fqbn esp32:esp32:lolin_s2_mini

# Monitor, C-c to exit
arduino-cli monitor -p /dev/ttyACM0
```

I do have sample `c_cpp_properties.json` and `C/C++ Runner (settings.json)` code and hope to have them applied soon. Without them I get the dreaded: `Squiggles are disabled for this translation unit`. I'll post them here once I get them working properly.

# Initial requirements

- Arduino enviroment
- ESP32
  - GPIO 15 - Red
  - GPIO 14 - Yellow
  - GPIO 13 - Green
  - GPIO 12 - Blue
  - GPIO 04 - Piezo (currently not used)
  - GPIO 01 - Debug ??? Still need to check this, just needs to be a digital output
- MQTT
  - accept command strings from the device/cmd topic
    - coomands are in the format of "color cmd"
      - where command is:
        - on
        - off
        - fflash (fast flash)
        - sflash (slow flash)
        - fade
      - where color is
        - red
        - green
        - yellow
        - blue
      - addition commands:
        - all off
        - all on
      - commands can be issued asynchronously
  - states can be monitored on device/state topic
- Flash fast
- Flash slow
- Fade on and off

I'll adjust these a bit later as I refine my ideas.

## Hardware

- 24v power supply
- 5v step down regulator
- ESP32 Lolin S2 Mini
- Quad MOSFET driver board
- PRE-F LED Andon Tower Light 56mm (I think)

This is stuff I have so I didn't make design decissions based on anything more than 'I've got it and I'm not using it". After many years of projects I have a lot of junk in my junk box.

## Mistakes I made:

- I threw this together in a few minutes and I hadn't really thought the whole thing through.
- I did't know how to communicate some of my requirements yet (like keep flashing or fading until an off has been issued) but I am learning (prompting)
- I've got a few things poorly organized. This isn't a project by my employer, this is a hobby project with junk I have on hand. So ... ;-)

## Lessons learned

- Prompting - better prompts, better responses
- Context engineering
- MCP/Agents
- Vibe coding, while I am using here I am/will be monitoring it closely. I'm experimenting and learning (the actual goal of this project).

I have to say that as of right now (2026) I'm still trying to absorb what all this means. Ive manage to grasp dribs and drabs and apply them to my VS Code use. I did setup Emacs but I'm not quite there yet on using AI in Emacs.

# Date

Sun Jan 25 10:29:29 AM EST 2026
