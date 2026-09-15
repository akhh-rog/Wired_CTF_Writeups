### The Sleepy Mechanic
Vehicle : Skoda Octavia A5 2011

Snapshot 1
Dashboard Engine Speed : 0 RPM

## Time        ID      Data
00.001        77E     0562220D5565AAAA
00.002        77E     0562220520AAAAAA
00.003        77E     0462220C65AAAAAA
00.004        77E     0462101484AAAAAA
00.005        77E     0562F40C0000AA55

Snapshot 2
Dashboard Engine Speed : 1000 RPM

## Time        ID      Data
00.101        77E     0562220D5465AAAA
00.102        77E     0562220521AAAAAA
00.103        77E     0462220C66AAAAAA
00.104        77E     0462101485AAAAAA
00.105        77E     0562F40C1388AA55

Snapshot 3
Dashboard Engine Speed : 1250 RPM

## Time        ID      Data
00.151        77E     0562220D5565AAAA
00.152        77E     0562220520AAAAAA
00.153        77E     0462220C67AAAAAA
00.154        77E     0462101486AAAAAA
00.155        77E     0562F40C186AAA55

Snapshot 4
Dashboard Engine Speed : 1500 RPM

## Time        ID      Data
00.176        77E     0562220D5465AAAA
00.177        77E     0562220521AAAAAA
00.178        77E     0462220C68AAAAAA
00.179        77E     0462101487AAAAAA
00.180        77E     0562F40C1D4CAA55

Snapshot 5
Dashboard Engine Speed : 2000 RPM

## Time        ID      Data
00.201        77E     0562220D5065AAAA
00.202        77E     0562220520AAAAAA
00.203        77E     0462220C68AAAAAA
00.204        77E     0462101487AAAAAA
00.205        77E     0562F40C2710AA55

Snapshot 6
Dashboard Engine Speed : 3000 RPM

## Time        ID      Data
00.301        77E     0562220D4065AAAA
00.302        77E     0562220521AAAAAA
00.303        77E     0462220C69AAAAAA
00.304        77E     0462101488AAAAAA
00.305        77E     0562F40C3A98AA55


The log contains 5 different CAN message types on ID 77E repeated across every snapshot

comparing the data fields across snapshots of varying engine speeds, line 5 (0562F40C...) is the only message where the inner bytes scale directly and linearly with the dashboard RPM readings.

Converting the signal bytes from Hexadecimal to Decimal across the snapshots reveals a simple linear relationship:
$$\begin{aligned} \text{1000 RPM} &\rightarrow \text{0x1388} = 5000_{10} \quad &(5000 / 1000 = 5) \\ \text{1250 RPM} &\rightarrow \text{0x186A} = 6250_{10} \quad &(6250 / 1250 = 5) \\ \text{1500 RPM} &\rightarrow \text{0x1D4C} = 7500_{10} \quad &(7500 / 1500 = 5) \\ \text{2000 RPM} &\rightarrow \text{0x2710} = 10000_{10} \quad &(10000 / 2000 = 5) \\ \text{3000 RPM} &\rightarrow \text{0x3A98} = 15000_{10} \quad &(15000 / 3000 = 5) \end{aligned}$$

raw value equal engine speed times 5

Calculating Target (2500 RPM)
2500x5=12500to10
Convert to 16-bit Hexadecimal:30d416

ssemble Final CAN Payload:Substitute 30D4 into the payload format 0562F40C____AA55 $\rightarrow$ 0562F40C30D4AA55.Format Flag:Combine with standard CAN ID prefix (77E#) $\rightarrow$ wired{77E#0562F40C30D4AA55}

this is all what i have 
