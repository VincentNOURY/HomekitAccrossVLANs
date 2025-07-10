# How to setup HomeKit accross VLANS

## Why is that an issue ?

In my case I have a network with separate VLANs for IOT, HomeAssistant and my LAN, this creates an issue for setting up HomeKit since the protocol is designed to be used on a single VLAN/subnet.

If you have the same issues as I did, read the following.

## Context and prerequisites

### The environment

I used OPNsense but any firewall with an mDNS repeater service should do the trick.<br>I'll explain the setup with 2 VLANs (HomeAssistant and LAN) but you can add as many LANs as you want/need.

## Prerequisites

- Homekit bridge setup on your HA instance. [Documentation](https://www.home-assistant.io/integrations/homekit/)
- mDNS reapeter service on you FW.

## Setting this up

### mDNS service

On OPNsense, just select the HomeAssistant and LAN interfaces and check Enable.<br>Nothing else, UDP broadcast relay is not needed.

### Firewall rules

You'll need two floating rules :
| Protocol | Interface            | Source                                                      | Source port | Destination            | Destination port |
|:--------:|----------------------|-------------------------------------------------------------|:-----------:|------------------------|:----------------:|
| UDP      | HomeAssistant<br>LAN | HomeAssistant Instance<br>LAN net (or specific LAN devices) | 5353        | 224.0.0.251/32         | 5353             |
| TCP      | LAN                  | LAN net (or specific LAN devices)                           | any         | HomeAssistant Instance | 21060-21070*     |

\* Note : The 21060-21070 range can be reduced to a single port if you expose only one bridge.<br>It will be fixed in HomeAssisant and you can find it in the name (ex: HASS Bridge:21062, runs on port 21062).

## Note for Docker users

If you use multiple networks (in particular a macvlan or ipvlan to assign a specific IP to HA), I found online some notes that Docker sends the packets only to the first network assigned (alphabetically), if this is the case, you can use the "name" option for your network and change it to a_network for example.
