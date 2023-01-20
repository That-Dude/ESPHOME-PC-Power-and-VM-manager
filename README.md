# ESPHOME PC Power and VM manager

 Simple ESPHOME project to control PC power and Turn ON/OFF Virtual manchines

# The Problem

I have a gaming PC installed with unRAID and two KVM virtual machines running
Windows 11 for VR gameing and Ubuntu for emulation, the Nvidia card is passed
through to which ever VM is running - this all works great.

This unRAID base PC is connected to my restricted managment VLAN and
Home-Assistant cannot 'see to' it from it's limited IoT VLAN. This is a pain
as there is no way for HA to turn ON/OFF the PC using WoL, or to control the
virutal machines as that would require SSH access - which is not availible
to HA on it's restricted VLAN.

# The Solution

I deciced to solve this problem by building an ESPHOME device that can turn
on the PC with a momentaly press of the power button conector, I've also
wired the motherboard LED to an input pin so I can see that the PC is
actually turn on.

To control the state of the virual machines was a little more involved. I
had a cheap joystick connector laying around so I wired the first two
buttons up to the ESPHOME deivce so I can simulate button presses which
are interpressed by a script on the unRAID server to switch between the
virutal machines.

<img width="642" alt="Home-Assistant-ESP-screenshot" src="https://user-images.githubusercontent.com/6509533/213774081-9dc68d25-2d79-43e7-9dbe-fec37419301b.png">

# Bill of materials

1. Zero Delay Arcade USB Encoder £4.50 (ebay)
2. Wemos D1 Mini clone £4.20 (amazon)
3. 4 x PC817C optocouplers £0.44
4. 4 x 1K resister £0.08
5. 3D printed case (optional)

As you can see, this was a very cheap solution, even if I didn't already have
all of the parts to hand.

# The Device

![esp-joystick-01](https://user-images.githubusercontent.com/6509533/213776653-9e587cf8-dbf8-4a27-a7dd-948465e5c27d.jpg)

![esp-joystick-02](https://user-images.githubusercontent.com/6509533/213776670-5d69f8b1-3476-4cb4-80a2-78fa3d34a81b.jpg)

![esp-joystick-03](https://user-images.githubusercontent.com/6509533/213776686-f0eb21c3-3cdb-4e9c-8a03-3b3386c6a6f7.jpg)

Try to overlook my terrible soldering :-)

From right to left:

Opto-coupler 1 is connected as a switch to GPIO D2 on the D1-Mini via a 1k
resistor. The oposite side os connected to the PC power button header.

Opto-coupler 2 is connected the PC power LED via a 1k resister. The other
side os connected directly to the D1-Mini GPIO D1 as a sensor input.

Opto-coupler 3 is connected as a switch to GPIO D5 on the D1-Mini via a 1k
resistor. The oposite side os connected to joystick controller button 1
socket.

Opto-coupler 3 is connected as a switch to GPIO D7 on the D1-Mini via a 1k
resistor. The oposite side os connected to joystick controller button 2
socket.

# ESPHOME yaml file

You will need to adjust this ESPHOME yaml to suit your setup, but this is
very simple.

```YAML
esphome:
  name: "mancave-esphome-gamepc"

esp8266:
  board: d1_mini

# Enable logging
logger:
#  level: VERY_VERBOSE

# Enable Home Assistant API
api:
  encryption:
    key: "/tQw2UOQmOqF8pDOFn9gS+G/7yLYoTG0940g3ZGOqF8="

ota:
  password: "7a555f60a13ecad11ed98e79d02aff4b"

wifi:
  ssid: !secret wifi_ssid
  password: !secret wifi_password

  # Enable fallback hotspot (captive portal) in case wifi connection fails
  ap:
    ssid: "Mancave-Gamepc2 Fallback Hotspot"
    password: "gtTJsUvF3OLk"

captive_portal:

# Webserver - no internet access required
web_server:
  local: true  
  auth:
    username: admin
    password: "justatest"

# ************************************** BUTTONS **************************************

# Create a series of buttons becuase they look better on the HA front end
button:
  - platform: template
    name: "Start EMU"
    id: button_emu
    icon: "mdi:controller"
    on_press:
      - logger.log: "Button 1 pressed"
      - switch.turn_on: b1_short_press

  - platform: template
    name: "Start VR"
    id: button_vr
    icon: "mdi:virtual-reality"
    on_press:
      - logger.log: "Button 2 pressed"
      - switch.turn_on: b2_short_press

  - platform: template
    name: "Power Button Toggle"
    id: power_button_short_press
    icon: "mdi:toggle-switch-outline"
    on_press:
      - logger.log: "PC power button - short press"
      - switch.turn_on: power_short_press

  - platform: template
    name: "Power Hard Shutdown"
    id: power_button_long_press
    icon: "mdi:toggle-switch-outline"
    on_press:
      - logger.log: "PC power on - long press"
      - switch.turn_on: power_long_press

# ************************************** SWITCHES **************************************

switch:
  - platform: gpio
    name: "emu"
    internal: true # hide from HA - we're using the button above to tigger this switch
    pin: D5   # game controller button 01
    id: b1_short_press
    inverted: no
    on_turn_on:
    - delay: 5ms
    - switch.turn_off: b1_short_press

  - platform: gpio
    name: "vr"
    internal: true # hide from HA - we're using the button above to tigger this switch
    pin: D7   # game controller button 02
    id: b2_short_press
    inverted: no
    on_turn_on:
    - delay: 5ms
    - switch.turn_off: b2_short_press

  - platform: gpio
    name: "powershortpress"
    internal: true # hide from HA - we're using the button above to tigger this switch
    pin: D2   # Power button output pin
    id: power_short_press
    inverted: no
    on_turn_on:
    - delay: 20ms
    - switch.turn_off: power_short_press

  - platform: gpio
    name: "powerlongpress"
    internal: true # hide from HA - we're using the button above to tigger this switch
    pin: D2   # Power button output pin
    id: power_long_press
    inverted: no
    on_turn_on:
    - delay: 3500ms
    - switch.turn_off: power_long_press

  - platform: template
    name: "bootstatus"
    internal: true # hide from HA - we're using the button above to tigger this switch
    id: switch_online_status
    turn_on_action:
    - binary_sensor.template.publish:
        id: sensor_boot_status
        state: ON
    - delay: 10s
    - binary_sensor.template.publish:
        id: sensor_boot_status
        state: OFF

  - platform: template
    name: "vmstatus"
    internal: true # hide from HA - we're using the button above to tigger this switch
    id: switch_vm_status
    turn_on_action:
    - binary_sensor.template.publish:
        id: sensor_vm_status
        state: ON
    - delay: 10s
    - binary_sensor.template.publish:
        id: sensor_vm_status
        state: OFF

# ************************************** BINARY SENSORS **************************************

binary_sensor:
  - platform: gpio
    name: "PC Power Status"
    pin:
      number: D1
      mode: INPUT_PULLUP
      inverted: true
    filters:
      - delayed_off: 30ms

  - platform: template
    name: "PC Boot status"
    id: sensor_boot_status

  - platform: template
    name: "VM Status"
    id: sensor_vm_status
```

