# B1nary — CTF Writeup

## Challenge Overview
* **Category:** Steganography / Misc
* **Difficulty:** Easy
* **Flag:** `wired{jU5t_@_3ina3y_dUmp1}`

## Description
The challenge provides a binary file `chall.txt` containing continuous bit data. Standard decoding reveals decoy flags (`wired{fake_flag}`) and misaligned ASCII noise due to bit-shifting.

## Solution
Running a bit-alignment shift script parses the 8-bit windows at a 1-bit offset to recover the accurate ASCII stream.

## flag wired{jU5t_@_3ina3y_dUmp1}
### Execution
Run the solver script in PowerShell:
```powershell
# Extract real flag from binary dump by shifting bit alignment
$raw = ([string](Get-Content chall.txt -Raw)) -replace '[^01]', ''

for ($shift = 0; $shift -lt 8; $shift++) {
    $b = $raw.Substring($shift)
    $chars = for ($i = 0; $i -le $b.Length - 8; $i += 8) {
        [char][convert]::ToInt32($b.Substring($i, 8), 2)
    }
    $text = -join $chars
    $matches = [regex]::Matches($text, 'wired\{[^}]+\}')
    foreach ($m in $matches) {
        if ($m.Value -notlike "*fake_flag*") {
            Write-Host "Found Flag (Shift $shift): $($m.Value)" -ForegroundColor Green
        }
    }
}



