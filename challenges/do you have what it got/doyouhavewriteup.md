## Do_you_have_what_it_got 
in this challange the author gave me a .pcap file which i opened in wireshark to manaully inspect and also using python in powershell to achive the flag
* wireshark
Protocol Analysis: Port 502 indicates Modbus TCP communication (a standard industrial control system protocol).
Payload/Hex Inspection:

Looking at Frame 2 (Packet 2):

The raw hex dump shows 0020  50 18 20 00 6f 58 00 00  4f 4b

The last two bytes translate in ASCII to OK (4f 4b).

Looking at the source ports on incoming Modbus requests from 192.168.10.10:

Packet 1: Source port 44160 (0xAC80)

Packet 3: Source port 54703 (0xD5AF)

Packet 5: Source port 43671 (0xAAA7)

Packet 7: Source port 52643 (0xCDA3)
 gave up on this cus it was too much i thought
 so went with python
 python3 -c '
import pyshark
cap = pyshark.FileCapture("file.pcap", display_filter="ip.src == 192.168.10.10 && tcp.dstport == 502")
data = bytearray()
for pkt in cap:
    if hasattr(pkt.tcp, "payload"):
        data.extend(bytes.fromhex(pkt.tcp.payload.replace(":", "")))
print(data.decode(errors="ignore"))
'

then errors came in
The tshark command is not in your Windows PATH variable
 
 since it was in a diff directory it hpnd

 python -c "import re; data=open(r'C:\Users\AMRITNATH\Downloads\Do_you_have_what_it_got.pcap', 'rb').read(); print([x.decode() for x in re.findall(rb'wired\{[^\}]+\}', data)])"
 Opened PowerShell and navigate directly to downloads

 Ran this command to search all .pcap or .pcapng files in that folder:
 python -c "import os, re; files=[f for f in os.listdir('.') if f.endswith(('.pcap', '.pcapng'))]; [print(f, re.findall(rb'wired\{[^\}]+\}', open(f, 'rb').read())) for f in files]"
 The output showed me  Do_you_have_what_it_got.pcap [], meaning the flag is not stored in plain ASCII text inside the PCAP; it is constructed from packet data across multiple packets.
 'to get tht another code
 python -c "import struct; data = open('Do_you_have_what_it_got.pcap', 'rb').read(); payloads = [b''.join([p[54+12:] for p in data.split(b'\x08\x00\x45\x00')[1:] if len(p) > 54 and p[12:16] == b'\xc0\xa8\x0a\x0a'])]; print('Flag:', ''.join(re.findall(r'[ -~]{4,}', data.decode('latin1'))))"

 then
 python -c "
with open('Do_you_have_what_it_got.pcap', 'rb') as f:
    d = f.read()

# Extract source ports of packets originating from 192.168.10.10 to port 502
# IP 192.168.10.10 is c0 a8 0a 0a
ports = []
idx = 0
while True:
    idx = d.find(b'\xc0\xa8\x0a\x0a', idx)
    if idx == -1: break
    # Check if destination IP is 192.168.10.20 or 192.168.10.30
    if idx + 8 < len(d) and d[idx+4:idx+8] in [b'\xc0\xa8\x0a\x14', b'\xc0\xa8\x0a\x1e']:
        src_port = int.from_bytes(d[idx+8:idx+10], 'big')
        dst_port = int.from_bytes(d[idx+10:idx+12], 'big')
        if dst_port == 502:
            ports.append(src_port)
    idx += 1

# Method 1: Convert high bytes of source ports to ASCII
flag1 = ''.join([chr(p >> 8) + chr(p & 0xff) for p in ports])
print('Source Port ASCII Extraction:', flag1)
"

again error
 The error occured cuz  line breaks inside double quotes when pasting a multiline script directly into PowerShell.
  so i ran it via a python file
  import re

with open('Do_you_have_what_it_got.pcap', 'rb') as f:
    d = f.read()

# Parse pcap packet structures for client 192.168.10.10 -> port 502
ports = []
idx = 0
while True:
    idx = d.find(b'\xc0\xa8\x0a\x0a', idx)
    if idx == -1: 
        break
    if idx + 12 < len(d) and d[idx+4:idx+8] in [b'\xc0\xa8\x0a\x14', b'\xc0\xa8\x0a\x1e']:
        src_port = int.from_bytes(d[idx+8:idx+10], 'big')
        dst_port = int.from_bytes(d[idx+10:idx+12], 'big')
        if dst_port == 502:
            ports.append(src_port)
    idx += 1

# Convert source ports to ASCII
chars = []
for p in ports:
    chars.append(chr((p >> 8) & 0xFF))
    chars.append(chr(p & 0xFF))

extracted = "".join(chars)
print("Extracted Text:", extracted)

# Search for flag pattern wired{...}
match = re.search(r'wired\{[^\}]+\}', extracted)
if match:
    print("FLAG FOUND:", match.group(0))

another error came cus it coudnt get the file
so i ran this

python -c "import re; d=open(r'C:\Users\AMRITNATH\Downloads\Do_you_have_what_it_got.pcap', 'rb').read(); print([s.decode('latin1') for s in re.findall(rb'[ -~]{6,}', d) if b'wired' in s or b'{' in s])"

got it as my first output (finally)
port parsing method extracted raw binary data because the source ports in this pcap are not raw ASCII characters.
flag is split across packets or needs payload concatenation, copy and paste this command into PowerShell:
python -c "
import re

with open(r'C:\Users\AMRITNATH\Downloads\Do_you_have_what_it_got.pcap', 'rb') as f:
    raw = f.read()

# Extract all ASCII strings longer than 5 characters
strings = re.findall(r'[ -~]{5,}', raw.decode('latin1'))

# Look for flag prefix
flag_matches = [s for s in strings if 'wired' in s or '{' in s]
print('Found strings:', flag_matches)
"
* The result Found strings: [] confirms that the flag string wired{...} is not stored in plain ASCII, nor hidden directly in the source ports.
ran  this PowerShell command to extract the TCP payload lengths and individual payload bytes:
python -c "
with open(r'C:\Users\AMRITNATH\Downloads\Do_you_have_what_it_got.pcap', 'rb') as f:
    d = f.read()

# 1. Check Modbus payload lengths (Len in Info column)
# Packets from 192.168.10.10: c0 a8 0a 0a
lens = []
payload_bytes = []
for i in range(len(d)-12):
    if d[i:i+4] == b'\xc0\xa8\x0a\x0a':
        # Grab TCP Data length or Modbus payload
        payload_len = d[i+12] if i+12 < len(d) else 0
        lens.append(payload_len)

# Try decoding payload lengths directly to ASCII
print('Decoded from Lengths:', ''.join([chr(x) for x in lens if 32 <= x <= 126]))

# 2. Extract single anomalous/non-routine packet payload
# Search for non-standard ASCII bytes inside payload sections
unique_chars = []
for i in range(0, len(d)-4):
    if d[i:i+6] == b'wired{' or d[i:i+5] == b'WIRED':
        print('DIRECT MATCH AT INDEX:', i, d[i:i+50])
"
The flag is assembled by decoding the Modbus TCP payload data (the payload lengths Len=14, 12, 8, 19, 11, 18... visible in my Wireshark list).

extracted the flag using Python without relying on low-level byte offsets, install scapy to parse the PCAP file correctly.

pip install scapy
python -c "
from scapy.all import rdpcap
import re

packets = rdpcap(r'C:\Users\AMRITNATH\Downloads\Do_you_have_what_it_got.pcap')
payloads = []

for pkt in packets:
    if pkt.haslayer('IP') and pkt.haslayer('TCP'):
        if pkt['IP'].src == '192.168.10.10' and pkt['TCP'].dport == 502:
            if pkt.haslayer('Raw'):
                payloads.append(bytes(pkt['Raw']))

combined = b''.join(payloads)
print('--- Extracted ASCII Payload ---')
print(combined.decode('latin1', errors='ignore'))
print('\n--- Flag Search Result ---')
print(re.findall(r'wired\{.*?\}', combined.decode('latin1', errors='ignore')))
"

--- Extracted ASCII Payload ---

Temperature=83Pressure=108Flow=737Motor=1273;Temp=175Valve=CLOSEDPressure=46Flow=787Flow=562Valve=CLOSEDPressure=100Tank=77;Valve=OPENFlow=335Valve=CLOSEDMotor=1410;Temp=125Pressure=68Motor=1328;Temp=163Pump=STOPPressure=85Motor=1356;Temp=177Motor=1530;Temp=250Flow=792MotorRPM=1489Valve=OPENPump=STOPFlow=864Valve=CLOSEDValve=OPENTank=86;Valve=CLOSEDMotor=1444;Temp=109Tank=56;Valve=OPENValve=OPENTank=37;Valve=OPENFlow=485MotorRPM=1202Pump=STOPTankLevel=87Temperature=50Flow=710;Pressure=55Pressure=75Pressure=30TankLevel=72Motor=1477;Temp=121MotorRPM=1592Flow=844;Pressure=75Pump=STOPPump=RUNMotorRPM=1715Flow=765Pump=STOPPump=STOPValve=CLOSEDTank=68;Valve=OPENFlow=559;Pressure=82MotorRPM=1461Temperature=153Tank=46;Valve=OPENMotor=1674;Temp=151Tank=8;Valve=CLOSEDPump=STOPValve=CLOSEDValve=CLOSEDTank=96;Valve=OPENPressure=49Valve=OPENPump=RUNMotorRPM=1614Motor=1772;Temp=206Flow=156;Pressure=114Tank=95;Valve=CLOSEDTankLevel=39Tank=37;Valve=CLOSEDFlow=326;Pressure=72Pump=STOPFlow=146Motor=1682;Temp=201Pressure=90Temperature=198Flow=297Temperature=170Tank=71;Valve=OPENMotor=1536;Temp=123Tank=45;Valve=OPENPressure=47Motor=1607;Temp=191Tank=29;Valve=CLOSEDMotor=1691;Temp=41Valve=OPENFlow=317;Pressure=79MotorRPM=1762Pump=RUNTemperature=66Temperature=207Pressure=64Tank=56;Valve=CLOSEDTank=95;Valve=CLOSEDValve=CLOSEDFlow=754;Pressure=28Flow=182Flow=172Pump=STOPFlow=550Pressure=33TankLevel=54Valve=OPENPressure=118Motor=1226;Temp=118Valve=OPENTankLevel=81MotorRPM=1485Flow=260;Pressure=60TankLevel=45Temperature=111Temperature=228Tank=58;Valve=OPENFlow=646Pressure=50Pump=STOPPressure=118Pump=RUNMotorRPM=1576Motor=1380;Temp=112Pressure=78Hacker_alert=77697265647B316E647535747269616C5F653570696E6F6167655F64657465637433647DValve=OPENMotorRPM=1418Motor=1313;Temp=53Motor=1754;Temp=40Flow=324;Pressure=100Flow=144Pump=STOPPump=RUNTank=81;Valve=OPENTank=11;Valve=CLOSEDTankLevel=67Flow=804;Pressure=36Pump=STOPTankLevel=17Flow=151;Pressure=36Tank=53;Valve=CLOSEDMotorRPM=1588Pump=RUNMotor=1510;Temp=41Temperature=163Flow=417;Pressure=99Valve=OPENTemperature=189Flow=518;Pressure=101MotorRPM=1653MotorRPM=1583Pump=STOPTemperature=167TankLevel=19Motor=1318;Temp=52Tank=20;Valve=OPENTemperature=44Valve=OPENFlow=460;Pressure=54Tank=75;Valve=CLOSEDValve=OPENFlow=550;Pressure=41Motor=1204;Temp=192Flow=833MotorRPM=1733Motor=1338;Temp=138Tank=5;Valve=CLOSEDPressure=97MotorRPM=1666Flow=255;Pressure=83MotorRPM=1396Temperature=111Pressure=67Motor=1519;Temp=98MotorRPM=1501MotorRPM=1406Tank=83;Valve=CLOSEDPump=RUNPressure=71Pump=RUNTank=15;Valve=OPENTank=73;Valve=OPENValve=CLOSEDValve=CLOSEDTank=47;Valve=CLOSEDFlow=233;Pressure=104Tank=79;Valve=OPENTankLevel=50Flow=162Motor=1501;Temp=220Valve=CLOSEDFlow=591;Pressure=64Tank=97;Valve=CLOSEDMotorRPM=1389Pressure=97Motor=1739;Temp=152Temperature=144Valve=OPENTemperature=240Temperature=237MotorRPM=1695Flow=296;Pressure=78TankLevel=75Pump=RUNFlow=131;Pressure=119TankLevel=29Temperature=170Motor=1452;Temp=184Flow=209Valve=OPENMotorRPM=1741Temperature=108Tank=57;Valve=CLOSEDTankLevel=14MotorRPM=1346Valve=OPENPump=RUNPump=RUNPressure=116Flow=429;Pressure=95Flow=172Motor=1330;Temp=107TankLevel=64Pump=STOPMotorRPM=1790TankLevel=90Tank=51;Valve=OPENFlow=360;Pressure=118TankLevel=21Pressure=28Temperature=217Flow=827;Pressure=61MotorRPM=1460Flow=483;Pressure=106Motor=1471;Temp=143Flow=883Flow=860;Pressure=79Pump=RUNPump=RUNPressure=101Motor=1674;Temp=148Pressure=103Flow=563MotorRPM=1718Flow=592Temperature=196MotorRPM=1559Pump=RUNTankLevel=69Tank=88;Valve=CLOSEDMotorRPM=1588Flow=728;Pressure=99Flow=440;Pressure=74Flow=460;Pressure=102Temperature=107MotorRPM=1494Tank=71;Valve=OPENMotor=1484;Temp=217Motor=1731;Temp=175Pump=STOPValve=CLOSEDPressure=37Temperature=138MotorRPM=1566Tank=97;Valve=CLOSEDMotorRPM=1529Pump=RUNMotorRPM=1294Tank=72;Valve=OPENFlow=595Flow=228;Pressure=80Valve=CLOSEDMotorRPM=1353Tank=18;Valve=OPENPressure=43Valve=CLOSEDMotor=1789;Temp=249TankLevel=55Pump=RUNFlow=491Flow=277Flow=350;Pressure=100Temperature=88MotorRPM=1779Pump=RUNPressure=102Valve=CLOSEDTankLevel=84Tank=37;Valve=CLOSEDTank=61;Valve=CLOSEDPressure=85Pump=RUNMotor=1676;Temp=66Temperature=79Pump=RUNValve=CLOSEDTemperature=209Pump=STOPFlow=397;Pressure=42MotorRPM=1507TankLevel=72TankLevel=53Flow=343Valve=OPENMotorRPM=1653Tank=50;Valve=CLOSEDTank=74;Valve=OPENTankLevel=32TankLevel=12Motor=1629;Temp=214Pump=STOPMotorRPM=1530Pump=RUNPump=STOPValve=OPENFlow=249;Pressure=72Pump=STOPTemperature=156Valve=CLOSEDMotorRPM=1560Pressure=37TankLevel=49Flow=517Flow=595;Pressure=101MotorRPM=1630Motor=1322;Temp=193Pump=RUNMotor=1261;Temp=209Pressure=120Flow=310MotorRPM=1459TankLevel=80Flow=401;Pressure=101Temperature=185TankLevel=86TankLevel=37Flow=174;Pressure=80Motor=1529;Temp=239MotorRPM=1293Tank=50;Valve=CLOSEDTank=27;Valve=CLOSEDPump=STOPMotorRPM=1230Tank=55;Valve=CLOSEDFlow=570Motor=1415;Temp=104Temperature=178Temperature=42Pressure=82Temperature=151Flow=406;Pressure=90TankLevel=60Tank=93;Valve=CLOSEDTemperature=181Flow=759;Pressure=50Pump=RUNMotorRPM=1294Motor=1380;Temp=96MotorRPM=1552Temperature=162Pressure=105Flow=314MotorRPM=1472MotorRPM=1495Motor=1349;Temp=177Tank=55;Valve=OPENValve=CLOSEDTemperature=80MotorRPM=1604Valve=CLOSEDMotorRPM=1537Motor=1430;Temp=74TankLevel=15MotorRPM=1356Motor=1523;Temp=47TankLevel=65Pressure=84Flow=192Tank=72;Valve=OPENFlow=809Tank=38;Valve=CLOSEDMotorRPM=1400MotorRPM=1269Pressure=86MotorRPM=1586TankLevel=53Tank=22;Valve=CLOSEDTemperature=197MotorRPM=1362TankLevel=7TankLevel=96Pressure=34Pressure=26Flow=530;Pressure=83MotorRPM=1513Valve=CLOSEDFlow=384;Pressure=35Pump=STOPMotor=1709;Temp=53TankLevel=83TankLevel=98Temperature=242Flow=202;Pressure=118Tank=71;Valve=OPENFlow=282Flow=353;Pressure=39Flow=611;Pressure=76Pressure=20Valve=OPENTankLevel=56Flow=780;Pressure=98Flow=181Pump=RUNPressure=113TankLevel=63Pump=RUNTank=18;Valve=OPENTankLevel=95Pump=RUNTankLevel=62MotorRPM=1417Pump=RUNPressure=96Pump=STOPFlow=896Pump=RUNMotor=1453;Temp=42MotorRPM=1348Tank=78;Valve=CLOSEDTank=21;Valve=OPENPump=STOPPump=RUNMotorRPM=1664Pump=STOPPump=RUNMotor=1423;Temp=131Temperature=240Valve=CLOSEDMotorRPM=1481Pump=STOPFlow=843Valve=CLOSEDTankLevel=37Pressure=104Motor=1352;Temp=197Motor=1496;Temp=220Tank=33;Valve=OPENMotorRPM=1494Flow=527Pressure=90Temperature=127Motor=1214;Temp=46Tank=41;Valve=OPENFlow=458Pressure=80Temperature=204Temperature=46Pump=RUNMotor=1270;Temp=92Pressure=42Tank=76;Valve=OPENFlow=328;Pressure=117Motor=1696;Temp=215Pressure=54Flow=597;Pressure=78Flow=318;Pressure=85Pressure=42Motor=1325;Temp=198MotorRPM=1373Valve=OPENTank=86;Valve=CLOSEDTankLevel=47Motor=1611;Temp=120TankLevel=38Pump=STOPPressure=69Pressure=119Pressure=64Flow=659Motor=1255;Temp=73TankLevel=21Valve=CLOSEDFlow=148;Pressure=95Flow=173Flow=495TankLevel=7TankLevel=26MotorRPM=1242Flow=758;Pressure=23Pressure=73Temperature=120MotorRPM=1637MotorRPM=1485MotorRPM=1521Flow=752;Pressure=58Temperature=208Flow=646Tank=14;Valve=CLOSEDPump=RUNFlow=693Valve=CLOSEDTemperature=209TankLevel=43Flow=808TankLevel=60Pressure=95MotorRPM=1739TankLevel=65Tank=27;Valve=CLOSEDMotor=1482;Temp=56Flow=286;Pressure=100Flow=759;Pressure=48Motor=1763;Temp=143TankLevel=13Valve=OPENMotorRPM=1646Pressure=23TankLevel=16Pump=RUNFlow=620;Pressure=104Flow=190Valve=CLOSEDPump=RUNFlow=772;Pressure=98Flow=651;Pressure=110Pump=STOPFlow=627Motor=1355;Temp=89Flow=301;Pressure=111MotorRPM=1786Valve=OPENTank=42;Valve=OPENPump=RUNPump=RUNValve=CLOSEDTankLevel=64TankLevel=46Flow=828;Pressure=76Pump=STOPValve=CLOSEDValve=CLOSEDMotor=1219;Temp=178TankLevel=6MotorRPM=1313Valve=OPENTank=5;Valve=CLOSEDPump=STOPTank=99;Valve=CLOSEDPump=STOPPump=STOPMotorRPM=1370Valve=OPENTank=87;Valve=CLOSEDMotor=1231;Temp=146Valve=CLOSEDPressure=118TankLevel=48Motor=1470;Temp=154Temperature=214MotorRPM=1291Flow=645Temperature=122Flow=818Motor=1768;Temp=99TankLevel=21Pressure=88Valve=CLOSEDTank=95;Valve=CLOSEDFlow=778;Pressure=65Valve=OPENTank=57;Valve=OPENPump=RUNValve=OPENTemperature=58Flow=833;Pressure=29Valve=OPENPressure=89Motor=1748;Temp=45Flow=694;Pressure=47Pressure=101Motor=1403;Temp=222Flow=383;Pressure=106Pump=STOPMotorRPM=1693Flow=415TankLevel=41Motor=1774;Temp=108Flow=130;Pressure=76TankLevel=91Pump=RUNFlow=898;Pressure=39Flow=356;Pressure=53MotorRPM=1358MotorRPM=1247Pressure=105Temperature=201Valve=OPENTankLevel=49Pump=RUNPressure=55Flow=140;Pressure=62Flow=238Valve=CLOSED 

## got the flag from this long ascii list
### Hacker_alert=77697265647B316E647535747269616C5F653570696E6O6167655F64657465637433647D
#### Converting the hexadecimal string (77697265647B316E647535747269616C5F653570696E6O6167655F64657465637433647D) into ASCII bytes reveals the flag:
##### wired{1ndu5tr1al_e5p1noage_detect3d}

