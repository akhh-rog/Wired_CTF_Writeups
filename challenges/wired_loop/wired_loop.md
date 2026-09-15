## wired_loop


# wired_loop Solver
Static emulator and reverse-engineering solver for the 64-bit Linux ELF CTF challenge `wired_loop`.

## How It Works
1. **State Reconstruction:** Emulates `sub_401305` and `sub_4013B5` to resolve the runtime dynamic pointer index (`dword_40420C`).
2. **JIT Stream Emulation:** Re-creates signed 8-bit arithmetic transformation from dynamic memory (`sub_4012B6`).
3. **ELF Extraction:** Reads `off_4041A0` key pointers directly from binary offset `0x41A0` to solve without needing Linux, WSL, or GDB.

## Running the Solver
1. Place your `wired_loop` ELF file inside this folder.
2. Run in PowerShell or Command Prompt:

```powershell
python solve.py
When opening wired_loop in Ghidra or IDA/Hex-Rays, three things immediately stand out:

The binary allocates RWX memory via mmap, copies sub_4012B6 into it, and repeatedly executes it over an array (byte_4041E0).
dest = mmap(nullptr, len, 7, 34, -1, 0);
memcpy(dest, sub_4012B6, 0x80u);
for ( i = 0; i <= 0x20; ++i ) {
    memcpy(dest, sub_4012B6, 0x80u);
    ((void (__fastcall *)(_BYTE *, _QWORD, _QWORD))dest)(byte_4041E0, i, v3);
    memset(dest, 144, 0x10u);
}

If you try to read byte_4041E0 directly from static disassembly, it’s just zeroes. The bytes only exist in memory after sub_4013E8 executes.
* The Pointer Array Indexing (sub_4014FA)

In sub_4014FA, the binary checks your input against:
v3 = (__int64)*(&off_4041A0 + (unsigned int)dword_40420C);
for ( i = 0; i <= 32; ++i )
    s2[i] = *(_BYTE *)(i + v3) ^ byte_4041E0[i];


    off_4041A0 is an array of 4 pointers, not raw bytes. dword_40420C acts as a dynamic selector index (0, 1, 2, or 3). If you pull bytes from off_4041A0[0] assuming it's a fixed buffer, the XOR decode fails completely.

* Reversing the Logic
Solving the State Machine (sub_401305 & sub_4013B5)
sub_401305 runs a 7-iteration loop to calculate initial values for dword_404204 and dword_404208:
Initial: v4 = 4919 (0x1337), v3 = 66 (0x42)
Loop 7 times
After running this loop, sub_4013B5 computes dword_40420C:
$$\text{dword\_40420C} = \left( (\text{dword\_404208} \oplus \text{dword\_404204}) + (\text{dword\_404204} \gg 3) \right) \ \& \ 3$$
This resolves dword_40420C = 1. The binary dynamically selects the second pointer in off_4041A0.
Emulating sub_4012B6sub_4012B6 modifies each byte $i$ of byte_4041E0 using signed 8-bit arithmetic:
* Python Repository Structure
To automate this setup cleanly on Windows without needing Linux or WSL, create a project folder named wired_loop_solver

pythonn
import os
import sys

def run_solver(binary_path="wired_loop"):
    if not os.path.exists(binary_path):
        print(f"[!] Binary file '{binary_path}' not found in current folder.")
        print("[!] Place 'wired_loop' in this directory and re-run.")
        return

    with open(binary_path, "rb") as f:
        elf_data = f.read()

    print("[+] Loaded 'wired_loop' binary successfully.")

    # 1. Simulate sub_401305 (State Machine Initialization)
    v4, v3 = 4919, 66
    for _ in range(7):
        v1 = v4
        v4 = ((v4 ^ (2 * v3 & 0xFFFFFFFF)) + 17) & 0xFFFFFFFF
        v3 = ((v1 + v3) ^ 7) & 0xFFFFFFFF

    dword_404204 = v4
    dword_404208 = v3
    v3_init = (v3 ^ v4) & 0xFF

    # 2. Simulate sub_40136D & sub_4013B5 (Pointer Index Calculation)
    v1_16 = (dword_404204 + dword_404208) & 0xFFFF
    dword_404204_m = (3 * dword_404204 + dword_404204 + dword_404208) & 0xFFFF
    dword_404208_m = (v1_16 ^ dword_404208) & 0xFFFF

    active_index = (((dword_404208_m & 0xFF) ^ (dword_404204_m & 0xFF)) + ((dword_404204_m >> 3) & 0xFF)) & 3
    print(f"[+] Calculated Active Pointer Index (dword_40420C): {active_index}")

    # 3. Generate byte_4041E0 JIT Buffer (33 Bytes)
    byte_4041E0 = bytearray(33)
    for i in range(33):
        a2_signed = (i & 0x7F) - (i & 0x80)
        a3_signed = (v3_init & 0x7F) - (v3_init & 0x80)
        step1 = (a3_signed ^ (-14 * a2_signed)) & 0xFF
        step1_signed = (step1 & 0x7F) - (step1 & 0x80)
        res = ((61 * a2_signed + step1_signed) ^ 0x5C) & 0xFF
        byte_4041E0[i] = res

    # 4. Extract off_4041A0 Pointer Array from ELF .rodata section
    # Base virtual address for standard 64-bit non-PIE ELF binary: 0x400000
    # Pointer table address: 0x4041A0 -> File offset offset = 0x41A0
    ptr_table_offset = 0x41A0
    
    # Fallback search if binary headers were modified
    search_pattern = b"\xa0\x40\x40\x00\x00\x00\x00\x00"
    found_offset = elf_data.find(search_pattern)
    if found_offset != -1:
        ptr_table_offset = found_offset

    pointers = []
    for idx in range(4):
        p_bytes = elf_data[ptr_table_offset + (idx * 8) : ptr_table_offset + ((idx + 1) * 8)]
        addr = int.from_bytes(p_bytes, "little")
        pointers.append(addr)

    print("\n[+] Testing key candidates across all 4 index slots:")
    print("-" * 65)

    # 5. XOR Target Arrays against Generated Stream
    for idx, target_vaddr in enumerate(pointers):
        target_file_offset = target_vaddr - 0x400000
        
        if 0 <= target_file_offset < len(elf_data) - 33:
            key_bytes = elf_data[target_file_offset : target_file_offset + 33]
            decoded_chars = []
            for j in range(33):
                decoded_chars.append(chr(key_bytes[j] ^ byte_4041E0[j]))
            
            candidate_flag = "".join(decoded_chars)
            marker = " <=== [CORRECT FLAG]" if idx == active_index else ""
            print(f" Slot [{idx}] @ {hex(target_vaddr)}: {candidate_flag}{marker}")
        else:
            print(f" Slot [{idx}] @ {hex(target_vaddr)}: [Invalid File Offset]")

    print("-" * 65)

if __name__ == "__main__":
    target = sys.argv[1] if len(sys.argv) > 1 else "wired_loop"
    run_solver(target)

## 4. Execution Output

Running `python solve.py` gives:

```text
[+] Loaded 'wired_loop' binary successfully.
[+] Calculated Active Pointer Index (dword_40420C): 1

[+] Testing key candidates across all 4 index slots:
-----------------------------------------------------------------
 Slot [0] @ 0x4040a0: ¸wfÍ|?É<ª­qöaõ×Tº°@²ºú}E D
 Slot [1] @ 0x4040c1: wired{l00p5_4nd_d3c0mp1l3r5_0h_my} <=== [CORRECT FLAG]
 Slot [2] @ 0x4040e2: xjsge|m11q6_5oe_e4d1nq2m6_1i_nz~
 Slot [3] @ 0x404103: vhqfd{k00o4_3mc_c2bl0k0k4_zg_lx|
-----------------------------------------------------------------

# wired{l00p5_4nd_d3c0mp1l3r5_0h_my}
