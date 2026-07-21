# Home Assistant on older hardware (Asus Seashell Eeepc 1005PE)
This repo aims at remembering the steps I followed to install Home Assistant on my old Asus Eeepc 1005PE laptop from 2010.  
The Intel Atom N450 processor is a x64 architecture, so the main issue to overcome was the lack of UEFI support that is now mandatory for HAOS, closely followed by the small amount of stock RAM (1GB).  
The RAM was an easy fix, I upgraded with a 2GB stick (maximum size compatible with this motherboard, sadly).  

## Installation
I followed the steps from this issue and aside from the resizing of the sda8 partition, it just worked: https://github.com/home-assistant/home-assistant.io/issues/39584  
(instead, I used gparted for all the steps related to partition manipulation and it went fine)  

## Useful information
HAOS version installed: haos_generic-x86-64-18.1  
