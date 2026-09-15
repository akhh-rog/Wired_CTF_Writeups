## The Final Broadcast CTF
for this challange we were given a chall.log file

The challenge description states mechanics searched through VINs/DTCs and were wrong. IDs 040 (VIN_OK) and 060 (ECU_OK) are noise frames.so we have to filter that out to get the flag took online resources to find this out

In CAN bus arbitration, lower arbitration IDs gain priority and transmit first. again used online tools to understand this 

then in that sorted list i tried converting the numbers from hex to ascii using online websites
Timestamp    ID     Data
--------------------------------------------------------
coverted to ascii---------()

00.001       510    336B776456 3kwdV
00.001       120    68346B335F3174---h4k3_1t
00.001       700    66516F3D fQo=
00.001       060    4543555F4F4B ECU_OK
00.001       620    795832307A yX20z
00.001       180    6E30745F793374---n0t_y3t
00.001       450    5A575237 ZWR7
00.001       500    597A527558 -- YzRuX
00.001       080    63346E5F3077     ---- c4n_0w
00.001       550    396F4D7A52------ no
00.001       040    56494E5F4F4B------no
00.001       321    3978375F3271-----9x7_2q
00.001       220    346C6D307374-----4lm0st
00.001       301    6B3333705F6730---k33p_g0
00.001       420    64326C79  -- d2ly

# combining all the fragments
c4n_0wh4k3_1tn0t_y3t4lm0stk33p_g09x7_2qd2lyZWR7YzRuX3kwdV9oMzRyX20zfQo=

c4n_0wh4k3_1tn0t_y3t4lm0stk33p_g09x7_2q is standard beacuse anything after tht is in the format of code64 decode so i decoded tht also got the flag from tht
d2lyZWR7YzRuX3kwdV9oMzRyX20zfQo=
converting then we get the flag
### wired{c4n_y0u_h34r_m3}


