# LEDTower
ESP32 controlling a R/Y/G/Bl LED Tower via MQTT

I'm no expert with the new code bots but I am learning. I do know how to write the code to accomplish this program. This is another experiment with ChatGPT 4, Bard and Huggin Face. So far I'm only accpting the ChatGPT generated code as the others are not correct. I'll experiment with different prompts and requiments to see if I can get the others working better. The requirements below are part of what I used in the first attempts.

# Initial requirements

- Arduino enviroment
- ESP32
  - GPIO 15 - Red
  - GPIO 14 - Yellow
  - GPIO 13 - Green
  - GPIO 12 - Blue
  - GPIO 04 - Piezo
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
  - states can be monitored on device/state
- Flash fast
- Flash slow
- Fade on and off

I'll adjust these a bit later as I refine my ideas.

## Mistakes I made:

- I threw this together in a few minutes and I hadn't really thought the whole thing trough.
- I don't know how to communicate some of my requriements yet (like keep flashing or fading until an off has been issued)
- I've got a few things poorly organized.

# Date

Tue 13 Jun 2023 22:16:57 PM EDT
