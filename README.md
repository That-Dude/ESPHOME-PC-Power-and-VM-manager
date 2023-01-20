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

# unRAID script

On the unRAID server I have a simple Bash script that starts on boot. It
connects to the HID device ID of the joystick and streams the commands into a
while loop. When a joystick button is pressed it stops any running VMs (if any)
and start the requested VM.

It also connected directly to the ESPHOME device and updates the the boot and
VM status.

This all works nicely in Home Assistant where I already have Alexa integrated.
I can now say "Alexa start vr" of "Alexa start emu", the TV turns on, the lights
adjust and the gaming experice begins.

...yeah I had a slow weekend :-)


```bash
#!/usr/bin/bash

# script to monitor button presses from a specific USB joystick, as defined by vifpid below.
#
# Requres hidapitester linux binary - there are macos and windows binaries too in case you need them.
# https://github.com/todbot/hidapitester
#
# My use case: I have built a simple ESP8266 deivce that can turn ON/OFF a PC over wifi - I'm using
# ESPHOME and HomeAssistant to automate this.
#
# I also wanted to send some command once the PC is powered up to launch different VMs so I added
# a cheap USB joystick controller to the project and activate it using the ESP8266.

# ***************************************** USER VARS ******************************************
hostname=mancave-esphome-gamepc.local
userpass="admin:justatest"
# This is the vid and pid of the joyside usb controller we are interacting with
vidpid="0079:0006"
update_online_status_delay=10
debug="ON"

# ***************************************** FUNCTIONS ******************************************
function_message() {

RED="\e[31m"
GREEN="\e[32m"
YELLOW="\e[33m"
ENDCOLOR="\e[0m"

  now=$(date +"%T")
  case "$1" in
  "info")
    echo -e "${now} ${GREEN}INFO:${ENDCOLOR} $2"
    ;;
  "debug")
    if [[ "$debug" == "ON" ]]; then
    echo -e "${now} ${YELLOW}DEBUG:${ENDCOLOR} $2"
    fi
    ;;
  "error")
    echo -e "${now} ${RED}ERROR:${ENDCOLOR} $2"
    ;;
  esac
}

function_debouce() {
    # after detecting a button press we read a few more input lines from the joystck to debounce the datastream
    read -r joystick_data_stream
    function_message debug "debounce joystick_data_stream = $joystick_data_stream"
    read -r joystick_data_stream
    function_message debug "debounce joystick_data_stream = $joystick_data_stream"
    read -r joystick_data_stream
    function_message debug "debounce joystick_data_stream = $joystick_data_stream"
}

function_vm_work() {
  vmtostart="$1"
  function_message debug "function_vm_work. vmtostart=${vmtostart}"
  vmstate=$(virsh domstate "$vmtostart")

  if [ "$vmstate" = "running" ]; then
    function_message info "$vmtostart is already running. Nothing to do!"
  else
    function_message info "$vmtostart is currently stopped."
    function_message info "Check if other VMs are running..."
    vmstat=$(virsh list --all)
    function_message debug "function_vm_work. vmstat=${vmstat}"

    if [[ "$vmstat" == *"pmsuspend"* ]]; then
      function_message info "There are VMs in power managment suspend state"
      function_message info "Waking then up so that we can then shut them down..."
      curl --digest --user ${userpass} -X POST "${IP}/switch/vmstatus/turn_on"
      # while loop to wait for vm to wake
      for i in $(virsh list | grep pmsuspend | awk '{print $2}'); do
        virsh virsh dompmwakeup "$i"
        sleep 5
      done
    elif [[ "$vmstat" == *"running"* ]]; then
      function_message info "INFO there are other VMs running."
      function_message info "Shuting down other VMs..."
      curl --digest --user ${userpass} -X POST "${IP}/switch/vmstatus/turn_on"
      virsh shutdown "emu"
      virsh shutdown "vr"
      sleep 10 # give vm's time to shutdown
    else
      curl --digest --user ${userpass} -X POST "${IP}/switch/vmstatus/turn_off"
      function_message info "INFO: No other VM's running"
      function_message info "INFO: starting $vmtostart..."
      virsh start "$vmtostart"
      curl --digest --user ${userpass} -X POST "${IP}/switch/vmstatus/turn_on"
    fi

  fi
}

# ***************************************** SCRIPT START ******************************************

function_message info "Resloving ${hostname} to an IP address"
IP=$(getent hosts $hostname | awk '{ print $1 }')
if [ $? -ne 0 ]; then
  function_message error "cannot resolve ${hostname}" 1>&2
  exit 1
fi
function_message info "IP = ${IP}"

# Tell the ESP device that this script is online
function_message info "Set computer boot status to ONLINE"
curl --digest --user ${userpass} -X POST "${IP}/select/boot_status/set?option=online"

# init timers
timer_start=$(</proc/uptime)
timer_start=${timer_start%%.*}                          # remove data after the dot
timer_start="$(printf '%d' "$timer_start" 2>/dev/null)" # convert string to int
function_message debug "timer_start = ${timer_start}"
timer_end=$((timer_start + 2))
function_message debug "timer_end = ${timer_end}"

# ****************************************** MAIN LOOP *******************************************

function_message info "Listening to joystick inputs"
while read -r joystick_data_stream; do

  # get a line of data from the joystick input data stream
  # delete the first 15 chars, we only need the tow chars at postiion 16/17
  button_press=${joystick_data_stream:15}

  case "$button_press" in
  "1F") # joystick button 1

    function_message info "Button 1 pressed - Starting EMU"
    #echo "INFO: Update VM boot status to ON"
    curl --digest --user ${userpass} -X POST "${IP}/switch/vmstatus/turn_on"
    function_vm_work "emu"
    function_debouce
    ;;
  "2F") # joystick button 2
    function_message info "Button 2 pressed - Starting VR"
   #echo "INFO: Update VM boot status to ON"
    curl --digest --user ${userpass} -X POST "${IP}/switch/vmstatus/turn_on"
    function_vm_work "vr"
    function_debouce
    ;;
  esac

  # periodically update the boot and 'vm running' status on the ESP device
  # it automatically sets itself to offline every n seconds
  if [ "$timer_start" = "$timer_end" ]; then
    function_message info "Update 'bootstatus' to ON"
    curl --digest --user ${userpass} -X POST "${IP}/switch/bootstatus/turn_on"

    vmstat=$(virsh list --all)
    if [[ "$vmstat" == *"running"* ]]; then
      function_message info "Update 'VM running' status to ON"
      curl --digest --user ${userpass} -X POST "${IP}/switch/vmstatus/turn_on"
    else
      function_message info "Update 'VM running' status to OFF"
      curl --digest --user ${userpass} -X POST "${IP}/switch/vmstatus/turn_off"
    fi

    # set new execution time
    timer_end=$((timer_start + update_online_status_delay))
    function_message debug "timer_end = ${timer_end}"
  fi

  timer_start=$(</proc/uptime)
  timer_start=${timer_start%%.*}                          # remove data after the dot
  timer_start="$(printf '%d' "$timer_start" 2>/dev/null)" # convert string to int

  # the next line pipes the output of the hidapitester command into the while loop
  # -q quite / -vidpid - the USB ID of my game controller / -l 6 - the number of bytes to read each pass.
done < <(./hidapitester -q --vidpid ${vidpid} -l 6 -t 10000 --open --read-input-forever)
```