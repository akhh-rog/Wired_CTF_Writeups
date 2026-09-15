# CTF Writeup: METRONOME Broken QR Code confusion

A writeup of a CTF challenge where a corrupted QR code  Python scripts which showed error after error and basic math worked
---

## 1. The Setup

The challenge gave me two things
1. A corrupted image file named `TheBrokenQR.png`.

**The Plan**: Fix the QR code, scan it  get the key submit the flag 


---

## 2. The attempts

### Failed Attempt 1: scanning it ?
I tried repairing row 6 and column 6 (the timing patterns) with Python scripts and manual drawing tricks. The image looked like a QR code so i scanned it got a thing? A hex string that looked like dis
   1e001b0c0d121a0a5d0736045a36000f3610591c360a5d0714

   idk i dont have a screen shot

### Failed Attempt 2: Command Prompt crashout
I tried running raw Python code inside Command Prompt without calling `python` first, getting hit with endless `'import' is not recognized` errors. Once the script finally ran (didnt worked as the flag was wrng)

---

## 3. The  Moment ig ? (using online help)


1. The ciphertext was encrypted with an **XOR cipher**.
2. XOR is completely reversible ($\text{Ciphertext} \oplus \text{Plaintext} = \text{Key}$).
3. The flag will have to start with `wired{`.

So, I lined up the first 6 hex bytes against `wired{` to see what key byte pops out:

* `0x1e` $\oplus$ `'w'` (`0x77`) = **`0x69`**
* `0x00` $\oplus$ `'i'` (`0x69`) = **`0x69`**
* `0x1b` $\oplus$ `'r'` (`0x72`) = **`0x69`**
* `0x0c` $\oplus$ `'e'` (`0x65`) = **`0x69`**
* `0x0d` $\oplus$ `'d'` (`0x64`) = **`0x69`**
* `0x12` $\oplus$ `'{'` (`0x7b`) = **`0x69`**

Every single character pointed to **`0x69`** (which is just the letter `'i'` in ASCII). The key wasn't some complex string—it was just the single byte `0x69` repeated over and over.

---

## 4. The Lazy Python Solution

Instead of overcomplicating image processing libraries, here is the tiny 4-line script that solved the entire thing in under a second:

hex_str = "1e001b0c0d121a0a5d0736045a36000f3610591c360a5d0714"
key_byte = 0x69

flag = "".join(chr(b ^ key_byte) for b in bytes.fromhex(hex_str))
print(flag)

flag was wired{sc4n_m3_if_y0u_c4n}

if i remember so

sorry if there is any mistake tho first time coder and a github user :/