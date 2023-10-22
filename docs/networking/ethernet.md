---
title: 'Ethernet'
date: 2021-08-25
tags: [ 'ethernet', 'RJ45', 'network' ]
---

There are two wiring standards used by RJ45 connectors, referred to as T568A
and T568B. The difference is the pin assignment for the green and orange pairs.
T568A pinouts are the most commonly used but either will work so long as both
ends of the cable are similarly wired.

|PIN |SIGNAL |T568A        |T568B        |
|--- |------ |------------ |------------ |
|1   |TX+    |White/Green  |White/Orange |
|2   |TX-    |Green        |Orange       |
|3   |RX+    |White/Orange |White/Green  |
|4   |TRD2+  |Blue         |Blue         |
|5   |TRD2-  |White/Blue   |White/Blue   |
|6   |RX-    |Orange       |Green        |
|7   |TRS3+  |White/Brown  |White/Brown  |
|8   |TRS3-  |Brown        |Brown        |

Cables with T568A wiring on one end and T568B on the other are known as
"crossover" cables. You can identify a crossover cable by comparing the order
of wires on each end. If the wires are the same on each end (regardless of
which pin configuration is used), it is a "straight-through" cable. If they are
different, it is a crossover cable.

Nowadays, most Ethernet switches and routers have a feature called auto-MDIX,
which can detect which type of port or cable (crossover or straight-through) is
connected and swap the transmit and receive pins accordingly, removing the need
for crossover wiring.

## Ethernet Categories

|Category |Max. Data Rate |Max. Bandwidth |Max. Distance          |Shielding |Usage                     |
|-------- |-------------- |-------------- |---------------------- |--------- |------------------------- |
|Cat. 1   |1 Mbps         |0.4 MHz        |                       |No        |Telephone and modem lines |
|Cat. 2   |4 Mbps         |4 MHz          |                       |No        |LocalTalk & Telephone     |
|Cat. 3   |10 Mbps        |16 MHz         |100 m                  |No        |10BaseT Ethernet          |
|Cat. 4   |16 Mbps        |20 MHz         |100 m                  |No        |Token Ring                |
|Cat. 5   |100 Mbps       |100 MHz        |100 m                  |No        |100BaseT Ethernet         |
|Cat. 5e  |1 Gbps         |100 MHz        |100 m                  |No        |100BaseT Ethernet, residential homes |
|Cat. 6   |1 Gbps         |250 MHz        |100 m  10Gb at 37 m    |Sometimes |10Gb at 37 m (121 ft.)|Gigabit Ethernet, commercial buildings |
|Cat. 6a  |10 Gbps        |500 MHz        |100 m                  |Sometimes |Gigabit Ethernet in data centers and commercial buildings |
|Cat. 7   |10 Gbps        |600 MHz        |100 m                  |Yes       |10 Gbps Core Infrastructure |
|Cat. 7a  |10 Gbps        |1000 MHz       |100 m  40Gb at 50 m    |Yes       |10 Gbps Core Infrastructure |
|Cat. 8   |25 Gbps (Cat8.1)40 Gbps (Cat8.2)|2000 MHz|30 m (98 ft.)|Yes       |25 Gbps/40 Gbps Core Infrastructure |

<p style="font-size: 12px" align="right">
    <a href="https://www.tripplite.com/products/ethernet-cable-types">Source</a>
</p>

## Shielded cable types

- **F/UTP – Foiled/Unshielded Twisted Pair**:  
Common in Fast Ethernet deployments, this cable will have a foil shield that wraps around unshielded twisted pairs.

- **S/UTP – Braided Shielding/ Unshielded Twisted Pair**:  
This cable will wrap a braided shield around unshielded twisted pairs.

- **SF/UTP – Braided Shielding+Foil/Unshielded Twisted Pairs**:  
This cable braids a shield around a foil wrap to enclose unshielded twisted pairs.

- **S/FTP– Braided Shielding/Foiled Twisted Pair**:  
This cable wraps a braided shield around all four copper pairs. Additionally, each twisted pair is enveloped in foil.

- **F/FTP-Foiled/Foiled Twisted Pair**:  
This cable encloses all copper pairs in foil. Additionally, each twisted pair is enveloped in foil.

- **U/FTP-Unshielded/Foiled Twisted Pairs**:  
This cable only envelopes the twisted pairs in foil.

- **U/UTP-Unshielded/UnshieldedTwisted Pair**:  
No sheathing is used. Standard Cat5e cable are examples of U/UTP cables.

## Shielding code:

- **TP**: Twisted Pair
- **U**: Unshielded or Unscreened
- **F**: Foil Shielding
- **S**: Braided Shielding

<p style="font-size: 12px" align="right">
    <a href="https://planetechusa.com/demystifying-ethernet-types-difference-between-cat5e-cat-6-and-cat7/">Source</a>
</p>

## Power over Ethernet (PoE)

|                           | PoE            | PoE+                | PoE++          | PoE++          |
|---------------------------|----------------|---------------------|----------------|----------------|
| IEEE-Standard             | 802.3af        | 802.3at             | 802.3bt        | 802.3bt        |
| PoE Type                  | Type 1         | Type 2              | Type 3         | Type 4         |
| Max. Power per Port       | 15,4 W         | 30 W                | 60 W           | 100 W          |
| Voltage Range (at Switch) | 44–57V         | 50-57V              | 50-57V         | 52-57V         |
| Max. Power to Device      | 12,95 W        | 25,5 W              | 51 W           | 71 W           |
| Voltage Range to Device   | 37-57V         | 42,5–57V            | 42,5–57V       | 41,1-57V       |
| Twisted Pairs Used        | 2-pair         | 2-pair              | 4-pair         | 4-pair         |
| Supported cable           | Cat3 or better | Cat5 or better      | Cat5 or better | Cat5 or better |
| Supported devices         | Access Points  | IP-Video telephones | Laptops        | TV sets        |
