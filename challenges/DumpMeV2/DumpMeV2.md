### DumpMeV2
The challenge DumpMeV2 on an ESP32 centers around hardware hacking, serial communication, and potentially extracting non-volatile storage data like NVRAM or flash memory.

for thsi challange i used arudino ide and installed various drivers needed for esp32

then i connected the board into my laptop which then allowed me to access what was inside of the esp32

it asked me various questions regarding flash memeory
The device interacts via a serial shell. Recognizing that persistent data survives power cycles on this platform points toward EEPROM/Flash emulation (`Commit` operation).

Since direct memory extraction or reading specific offsets via the command line was constrained, an Arduino script was flashed to read and scan the emulated EEPROM space.

after all this questions and hints i understood tht the esp32 had the flag at the eeprom

so in order to access the flag i used a c++ code which was then dumped into the esp32 
#include <EEPROM.h>

#define EEPROM_SIZE 512

void setup() {
  Serial.begin(115200);
  delay(1000);
  
  EEPROM.begin(EEPROM_SIZE);
  Serial.println("\nScanning ESP8266 EEPROM for persistent data...");

  String currentString = "";
  
  for (int i = 0; i < EEPROM_SIZE; i++) {
    char c = (char)EEPROM.read(i);
    
    if (c >= 32 && c <= 126) {
      currentString += c;
    } else {
      if (currentString.length() >= 3) {
        Serial.print("Offset ");
        Serial.print(i - currentString.length());
        Serial.print(": ");
        Serial.println(currentString);
      }
      currentString = "";
    }
  }
  
  Serial.println("Scan complete.");
}

void loop() {}


**Decoding the Payload:**
   Scanning the memory offset revealed a Base64 encoded string:
   
   RETRYd2lyZWR7ZWVlcHJvbV9zdXJ2aXZlc19mbGFzaH0=

we know that d2ly is a standard format for this type of text so i put it into decode 64 and removed extra char before it and it got me the flag

##  flag = wired{eeeprom_survives_flash}

done :)
