# Nexie

## Nexie clock on esp32
## Goal of this project: Make a Nexie tube clock with modern features and time sync via radio station DCF77.
Progress:
##### Idea ✅ 
##### Block Diagram ✅ 
##### SCH & PCB 🕓60%
#### HW fast test Micropython script:
```
import os
import math
import struct
import time
import random
from machine import I2S, Pin
import machine, neopixel

def make_tone(rate, bits, frequency):
    samples_per_cycle = rate // frequency
    sample_size_in_bytes = bits // 8
    samples = bytearray(samples_per_cycle * sample_size_in_bytes)
    volume_reduction_factor = 512
    amplitude = pow(2, bits) // 2 // volume_reduction_factor

    format = "<h" if bits == 16 else "<l"

    for i in range(samples_per_cycle):
        sample = int(amplitude * math.sin(2 * math.pi * i / samples_per_cycle))
        struct.pack_into(format, samples, i * sample_size_in_bytes, sample)
    
    return samples

def random_color():
    return (random.randint(0, 255), random.randint(0, 255), random.randint(0, 255))

# ======= BOARD CONFIGURATION =======
SCK_PIN = 32 # I2S MAX98357A
WS_PIN = 25 
SD_PIN = 33
I2S_ID = 0
n = 6  # number of LEDs
p = 5   # GPIO pin LED DAT
# ======= AUDIO CONFIGURATION =======
SAMPLE_SIZE_IN_BITS = 16
FORMAT = I2S.MONO
SAMPLE_RATE_IN_HZ = 44000
BUFFER_LENGTH_IN_BYTES = 2000
# ======= AUDIO CONFIGURATION =======
audio_out = I2S(
    I2S_ID,
    sck=Pin(SCK_PIN),
    ws=Pin(WS_PIN),
    sd=Pin(SD_PIN),
    mode=I2S.TX,
    bits=SAMPLE_SIZE_IN_BITS,
    format=FORMAT,
    rate=SAMPLE_RATE_IN_HZ,
    ibuf=BUFFER_LENGTH_IN_BYTES,
)

np = neopixel.NeoPixel(machine.Pin(p), n)

print("==========  START PLAYBACK ==========")

try:
    while True:
        # Random freq
        tone_freq = random.randint(1000, 2000)
        samples = make_tone(SAMPLE_RATE_IN_HZ, SAMPLE_SIZE_IN_BITS, tone_freq)
        print(f"Playing tone: {tone_freq} Hz")

        # Random colors
        for i in range(n):
            np[i] = random_color()
        np.write()

        # Playback
        t_start = time.ticks_ms()
        while time.ticks_diff(time.ticks_ms(), t_start) < 150:
            audio_out.write(samples)

except (KeyboardInterrupt, Exception) as e:
    print("caught exception {} {}".format(type(e).__name__, e))
# cleanup
audio_out.deinit()
print("Done")
```
