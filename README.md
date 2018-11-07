# tplink-msg
Understanding the communication protocol between Qualcomm (QCA) and Broadcom (BCM) cpu in some TP-Link modems.  
Usually the QCA sends a packet and then BCM answers back  
  
Tested in TP-Link Archer D7 and TP-LINK Archer D50  
  
TP-Link Archer VR2600v should use the same protocol (needs confirmation)  
TP-Link Archer D5 *v1* should be an Archer D7 rebranded (needs confirmation)  


***HEAVILY WIP***


## *1* Payload structure
```
PAYLOAD:

0X00 (1 byte)
Device sending data

Values:
	10: BROADCOM (BCM)
	11: QUALCOMM (QCA)



0X01 (1 byte)
Link status (usually BCM repeats what the QCA sends)

Values:
	01: WRITE
	02: IDLE
	03: ?
	07: ?
	09: ?



0X02 (2 byte)
First part of the counter?



0X04 (2 byte)
Counter. Everytime QCA sends a packet, this value is increased by 1. 
Eg: QCA sends packet 0, BCM sends packet 0; QCA sends packet 1, BCM sends packet 1



0X06 (1 byte)
Always 00?



0X07 (1 byte)
???

Values:
	IDLING
		QCA: 00
		BCM: 37

	WHEN QCA WANTS TO EDIT CONFIG:
		QCA: 04
		BCM: 04



0X08 (2 byte)
Random?



0X0a (4 byte)
Edit configuration xDSL data

Values:
	ALWASYS 0 in IDLING

	QCA SENDS: 
		00 00 01 01	T1.413		ANNEX-A		BITSWAP ON	SRA ON
		00 00 00 01	T1.413		ANNEX-A		BITSWAP OFF	SRA ON
		00 00 00 00	T1.413		ANNEX-A		BITSWAP OFF	SRA OFF

		02 00 00 00	G.lite		ANNEX-A		BITSWAP OFF	SRA OFF

		01 00 00 00	G.dmt		ANNEX-A		BITSWAP OFF	SRA OFF

		03 00 00 00	ADSL2		ANNEX-A		BITSWAP OFF	SRA OFF

		04 03 00 00	ADSL2+		ANNEX-A/L	BITSWAP OFF	SRA OFF
		04 00 00 00	ADSL2+		ANNEX-A		BITSWAP OFF	SRA OFF
		04 02 00 00	ADSL2+		ANNEX-M		BITSWAP OFF	SRA OFF
		04 04 00 00	ADSL2+		ANNEX-A/L/M	BITSWAP OFF	SRA OFF

		05 00 00 00	Auto		ANNEX-A		BITSWAP OFF	SRA OFF
		05 00 01 01	Auto		ANNEX-A		BITSWAP ON	SRA ON



0X0e (1 byte)
?

Values:
	QCA:
		ALWAYS 00.
		
	BCM:
		After IDLING: 02. 
		Otherwhise 00.



0X0f (1 byte)
?

Values:
	Always 0 in IDLING
```
  
## *2* Sample packets between QCA e BCM  
### Packet from QCA:
```
0000   01 02 03 04 05 06 f1 f2 f3 f4 f5 f6 81 00 00 01
0010   88 b5 11 02 00 00 09 6a 00 00 a0 51 00 00 00 00
0020   00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
0030   00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00

Ethernet stuff:
010203040506	f1f2f3f4f5f6	8100		000188b5 
DEST MAC	SRC MAC         TYPE (802.1Q)	VLAN 1

PAYLOAD:
11 02 0000 096a 00 00 a051 00000000 00 00 

Other:
000000000000000000000000000000000000000000000000000000000000
```
### Packet from BCM:
```
0000   f1 f2 f3 f4 f5 f6 01 02 03 04 05 06 81 00 00 01
0010   88 b5 10 02 00 00 09 6a 00 37 56 00 00 00 00 00
0020   02 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
0030   00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
0040   00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
0050   00 00 00

Ethernet stuff:
f1f2f3f4f5f6	010203040506	8100		000188b5
DEST MAC	SRC MAC		TYPE (802.1Q)	VLAN 1

PAYLOAD:
10 02 0000 096a 00 37 5600 00000000 02 00 

Other:
00000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000
```

