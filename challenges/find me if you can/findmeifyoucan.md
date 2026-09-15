## find_me_if_you_can
in this challange the author gave me a .pcap file so i used wireshark for solving this along with python in my powershell
Opened the .pcap file in Wireshark.

Applied a display filter in the top filter bar to view specific TTL values or IP headers ( ip.ttl = 64).

Inspected the packet details pane under Internet Protocol Version 4 to observe the specific TTL values packet by packet
looked at the time to live value 
and apllied a filter above to filter out 
i first tried the ip.ttl = 64 and it was the wroung set of packets so i went for
ip.ttl == 23
bcoz it was the most common and when subtracted from 64 gives a 41 value whicwhich which matches the challenge description: "takes the same path... except a handful of readings that seem to have come from a little further away."
clicked on the frist packet #5 and 7 and 25 all

Looked at the bottom-right pane (ASCII text area).

Notice the parameters in the payload

then intstead of using the manual method online tools suggested me to use python and scapy
"C:\Program Files\Wireshark\tshark.exe" -r Find_me_if_you_can.pcap -Y "ip.ttl == 23" -T fields -e text

with open("Find_me_if_you_can.pcap", "rb") as f:
    content = f.read()

# Look for the flag pattern inside raw binary bytes
import re
match = re.findall(b"wired\{[^\}]+\}", content)
for flag in match:
    print(flag.decode("utf-8"))
     
gave me file not found error so i ran it again at corrct path
 the decoded things is added to this 
 The numbers after the forward slash / at the end of each line are decimal ASCII values encoding the flag

 i put my extracted string/list of numbers directly into Python to print the flag
 numbers = [119, 105, 114, 101, 100, 123, 116, 116, 49, 95, 119, 104, 49, 115, 112, 51, 114, 115, 95, 116, 104, 51, 95, 115, 51, 99, 114, 51, 116]

# Convert ASCII values to string
flag = "".join(chr(n) for n in numbers)
print(flag)

## flag = wired{tt1_wh1sp3rs_th3_s3cr3t}
