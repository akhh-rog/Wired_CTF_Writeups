## ESP-ionage CTF 

Connected your laptop or mobile device to the Wi-Fi network named ESP-ionage.

Opened browser and navigated to [http://192.168.4.1](http://192.168.4.1)
inspected the website and found this 
<img src="/assets/img/ghost.png" alt="missing ghost"> along with the hint "Sometimes, what you see is not all there is."

Clicked the Headers sub-tab on the right side anto inspect the Response Headers to see if the flag is stored in a custom header 

under the sources ir smthg i  Got a Hint
Z28gdG8gL2dob3N0
encoded in base64
it yields:

go to /ghost

so i changed the url 
http://192.168.4.1/ghost](http://192.168.4.1/ghost
to thsi which then led me to another website which 
when inspected under the sources tab
<span style='display:none'>vhqdc</span>
<span style='display:none'>{uzkzq</span>
<span style='display:none'>_lnqfgtkhr</span>
<span style='display:none'>_zkk_ltrs_chd}</span>

Assembling these four hidden text spans in order gives the cipher string:
vhqdc{uzkzq_lnqfgtkhr_zkk_ltrs_chd}
tried adding and subtracting the letters and changing postion but then this clicked

Each character is shifted forward by 1 position (a ROT-1 / +1 Caesar shift).

Applying a +1 shift to the entire cipher string:


## FLAG  wired{vlaar_morghulis_all_must_die}

(valar dohaeris)

