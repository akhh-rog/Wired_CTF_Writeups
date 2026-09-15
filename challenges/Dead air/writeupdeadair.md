## Dead Air - CTF Writeup
## 🛠️ Tools Used
* **Universal Radio Hacker (URH):** Visual inspection, frequency offset tuning, and signal demodulation.
* **Python (`lzma` module):** Decompressing the `.xz` archive without high memory usage.

## i got a compressed IQ signal file called movie.cfile.xz and had to figure out what movie the hidden audio was from

**  The first hurdle was just getting the raw .cfile out of the .xz archive. I didn't have tar set up in my terminal, so I had to get creative and use a Python lzma one-liner to decompress it **

* ** first error 
EOFError: Compressed file ended before the end-of-stream marker was reached
It looked like the file was truncated or the download got cut off. But when I ran ls, I saw it had successfully extracted about 27MB of the file

# Step 2: Ditching Python for URH
I tried to write a whole Python script using numpy and scipy to demodulate the FM signal. It was a nightmare. The audio was super high-pitched, screeching, and playing way too fast because of sample rate mismatches.


To avoid killing myself , I ditched the Python DSP scripts completely and loaded my extracted file straight into Universal Radio Hacker (URH)

I opened movie.cfile in URH and set the format to Complex Float.

Switched over to the Demodulated Domain tab.

Selected FM from the modulation dropdown.

i changed the frquency or smthg and i got multiple mp3 files 

i used my own sleep deprived brain to find the dialouge from the audio file due to addiction of watching films on movies now 
 from there onwards it was a run for the flag then i got it finally 

 (kool challenge gps loved it)
 ## output file at 250k was the sweetspot for me
 ## flag = wired{ru5h_h0ur_3}
 
 after feeling bored i again tried this using python and these were my findings



>>>

# 1. Load IQ data
iq_data = np. fromfile('movie.cfile', dtype=np. complex64)

# 2. Basic FM Demodulation
angle = np.angle(iq_data[1:] * np.conj(iq_data[ :- 1]))

# 3. Helper to filter out high-pitch screeching
def lowpass_filter(data, cutoff=4000, fs=48000):
nyq = 0.5 * fs
normal_cutoff = cutoff / nyq
b, a = butter(5, normal_cutoff, btype='low', analog=False)
return lfilter(b, a, data)

# 4. Try common lower sample rates
rates_to_test = [250000, 192000, 96000, 48000]

for sr in rates_to_test:
# Resample to 48kHz target audio rate
audio = resample_poly(angle, 48000, sr)

# Filter out high-frequency screech/noise above 4kHz
audio_filtered = lowpass_filter(audio, cutoff=4000, fs=48000)

# Normalize volume
audio_filtered = audio_filtered / np.max(np. abs(audio_filtered))

filename = f'output_{sr//1000}k.wav'
wavfile.write(filename, 48000, (audio_filtered * 32767).astype(np. int16))
print(f"Saved {filename}")

Saved output_250k . wav
Saved output_192k . wav
Saved output_96k . wav
Saved output_48k . wav

---
