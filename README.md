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

