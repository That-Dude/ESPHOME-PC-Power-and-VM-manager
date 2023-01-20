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
