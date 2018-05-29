# Ultrasonic communication by STM32L4's DSP and MEMS microphone

![HelloWorld](./doc/HelloWorld_Spectrogram.jpg)

![S](./doc/Chirp_Spectrogram_S.jpg)

## Preparation: STM32L4 platform and FFT test code on MEMS mic

This project uses STM32L476RG as MCU and MP34ST01-M as MEMS microphone.

![platform](./doc/MEMSMIC_expansion_board.jpg)

![architecture](https://docs.google.com/drawings/d/e/2PACX-1vR1KKp2QeL_SmrnUsTl5zcwddQToPJmnSBHFnxiw78y3_3mjA7EzNl2iNcUA5aOW_jRAQapTNji-eJ7/pub?w=2268&h=567)

==> [Platform](PLATFORM.md)

==> [Test code](./basic)

## Ultrasonic communications experiment (FSK modulation)

==> [Experiment](EXPERIMENT.md)

==> [Test code](./ultracom)

Conclusion: the method (sort of FSK modulation) work very well in a silent room, but did not work in a noisy environment such as a meeting room. I have to come up with another approach, such as spread spectrum.

## Knowles MEMS mic

![Knowles](./doc/Knowles.jpg)

I have bought [this MEMS mic](http://akizukidenshi.com/catalog/g/gM-05577/): Knowles SPM0405HD4H. The spec is similar to the mic on the expansion board from STMicro. Although this one does not support ultrasonic, it should be OK.

## Chirp modulation experiment

### Two kinds of noises

I observed two kinds of noises in a room:

- Constant noises at specific frequencies: noises from motors/inverters???
- Bursty noises in a short period: cough, folding paper etc.

I _guess_ Chirp modulation might be suitable for ultrasonic communications in a noisy environment. No proof yet.

### Chirp modulation

Spectrum is spread out like Mt. Fuji:

![Chirp](./doc/Chirp.jpg)

### Chirp de-modulation

All the frequencies appear in one TQ(Time Quantum). I used [Audacity](https://www.audacityteam.org/) to capture the spectrogram:

![Chirp_Spectrogram](./doc/Chirp_Spectrogram.jpg)

Reference: [Chirp compression (Wikipedia)](https://en.wikipedia.org/wiki/Chirp_compression)

```
If a chirp sequence is a(n) and that for the compression filter is b(n), then the compressed pulse sequence c(n) is given by

c1(n)=IFFT[FFT{a(n)}*FFT{b(n)}]
```

Other references:
- [Chirp A New Radar Technique](http://www.rfcafe.com/references/electronics-world/chirp-new-radar-technique-january-1965-electronics-world.htm)
- [Radar Pulse Compression](https://www.ittc.ku.edu/workshops/Summer2004Lectures/Radar_Pulse_Compression.pdf)

### DFSDM setting

|Parameter    |Value/setting|
|-------------|-----|
|System clock |80MHz|
|Clock divider|25 (3.2MHz over-sampling)|
|Decimation   |32   |
|Filter       |sinc3|
|Sampling rate|100kHz|

### FFT setting

|Parameter    |Value/setting|
|-------------|-----|
|DMA interrupt|2048 samples/interrupt|
|SFFT         | TBD |

### Time Quantum (TQ)

1/100kHz * 2048samples/interrupt = 20.5 msec

### Frame (tentative)

"Start of frame" is to detect the beginning of transmission and also for frame synchronization with the transmitter.

```
Segment length: TQ[msec] = 20.5msec

Start of frame: 5TQ length
Bit: 3TQ length
End of frame: 5TQ length

Frame (656msec)
<- SOF    ->      <- Bit 0  ->   <- Bit 7  -><- EOF    ->
[S][S][S][S][Void][B0][B0][B0]...[B7][B7][B7][E][E][E][E]
    82msec          61.5msec       61.5msec     82msec

 ----------        ----------                 
             ----                 ----------  ----------
                            
Start of Frame
1: Chirp

End of frame
0: No chirp

Bit value
0: No chirp
1: Chirp

Void
0: No chirp
```

[Example] Ascii "S" character code (0x53)

![S](./doc/Chirp_Spectrogram_S.jpg)

### Transmission speed

It is quite slow! I will optimize each parameters to attain faster bit rate.

8bits * 1000(msec) / 656(msec) = 12bps

### FFT output from STM32L4 DSP with MEMS mic

I used a very cheap mic and [Jupyter Notebook to see the output](./agent/chirp_experiment/chirp.ipynb).

Sweep range: 16000Hz - 16000Hz

![16000](./doc/FFT_Chirp_16000.jpg)

Sweep range: 16000Hz - 17000Hz

![16000_17000](./doc/FFT_Chirp_16000_17000.jpg)

Sweep range: 16000Hz - 18000Hz

![16000_18000](./doc/FFT_Chirp_16000_18000.jpg)

### Chirp expriment on May 29, 2018

Sweep range 16000Hz - 18000Hz seemed to show the best result.

==> [Test code](./chirp)

## Devices I used in the past

This doppler sensor uses Radar to detect motion of people: https://github.com/araobp/motion-detector/blob/master/doc/adc_doppler.jpg.

Other interesting links:
- [CSS modulation](https://home.zhaw.ch/~rumc/wcom2/unterlagen/wcom2chap3CSS.pdf)
- [ASR (NEC)](https://www.nec.com/en/global/solutions/cns-atm/surveillance/asr.html)
- [ASNARO-2 (NEC)](https://www.nec.com/en/global/solutions/space/satellite_systems/nextar.html)
