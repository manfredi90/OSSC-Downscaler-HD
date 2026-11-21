OSSC Downscaler HD firmware
===========================

An unofficial secondary firmware for the [Open Source Scan Converter](https://github.com/marqs85/ossc)
that converts **HD and 31kHz sources on the VGA or component inputs** to
**1080i or 540p on HDMI**, for HD CRT TVs and monitors (through
HDMI, an HDMI→component converter or an HDMI→VGA DAC). It is useful for
multisync monitors that go up to 1080i at most, and for Sony HD CRTs, which
are lag-free only with 540p sources. The output is frame
locked to the input: no frame buffer, no added frame of lag.

It is the HD sibling of the [OSSC Downscaler](https://github.com/manfredi90/OSSC-Downscaler)
(15kHz 240p / 480i output). Both run from the OSSC's secondary firmware slot
("Launch 2nd FW"), so only one of the two can be installed at a time; neither
touches the official firmware: power cycling the OSSC always boots the
official firmware again.


> **Unofficial, experimental and AI.** This project was made with the help of AI. The code was written together with Claude. It's an unofficial fork of the original OSSC firmware.
> Use at your own risk. I am not responsible for any damage to the OSSC or the monitor


Thanks
------

This project was only possible because the OSSC project is an incredible project and has confirmed its versatility over the years. Thanks to Markus Hiienkari, all the credit goes to him.


Safety
------

* **31–34kHz output only.** Three independent layers keep the output in range:
  1. every output mode is computed for the 1080i line rate (33.75kHz at
     60Hz, 33.72kHz at 59.94Hz);
  2. anything the firmware cannot convert (no sync, unsupported input,
     out-of-range result) shows an internal test pattern with the same timing;
  3. a hardware watchdog in the FPGA, independent of the CPU, blanks video and
     all syncs if the output line rate leaves 31.0–34.5kHz.
* **The official firmware is never modified.** The image only fits the
  secondary slot (flash 0x80000–0xEFFFF). This firmware's settings live in a
  single sector of its own (0xF0000) and it never reads or writes the official
  profiles, its SD card profiles or the primary image.


Requirements
------------

* OSSC board (tested on rev 1.8) with the official firmware (tested with 1.22)
  as the primary firmware
* Source on the VGA input (AV3) or on a component input (YPbPr)
* A display that takes 1080i (or 540p) at the standard timing:
  * HD CRT TVs and monitors
    through an HDMI→component converter (tested: Sony HD CRT, generic HDMI to
    YPbPr converter, JVC DT series monitor);
  * RGB / RGBS HD monitors through an HDMI→VGA DAC; composite sync can be
    sent on the HSYNC line if the monitor needs it ("CSync on HSYNC").
* 1080i is the standard HDMI format and works everywhere; 540p is not a
  standard HDMI format: HD CRTs and most converters take it, many flat-panel
  TVs don't (or show it with artifacts at the edges).


Supported inputs and outputs
----------------------------

All outputs are 1920x540 per field / frame, on the 1080i line timing; the HDMI
active area is always the full 1920x540 window (black around smaller pictures).

| Input (60Hz)        | Output | Notes |
|---------------------|--------|-------|
| 1920x1080p          | 540p / 1080i | 540p: every output line is the average of 2 input lines (or the first of the two, without line blending); 1080i: each field takes every other line (optional flicker filter). All 1920 pixels |
| 1280x720p           | 540p / 1080i | 4 input lines become 3 output lines, weighted (540p: by area; 1080i: interpolated, the two fields 2/3 of an input line apart; without line blending: the nearest lines). The ADC samples at 1.5x: 1920 pixels |
| 1920x1080i          | 540p | every field becomes a 540p frame: the lines of the two fields are interpolated onto the same output lines (no bobbing). Also with "Output mode" 1080i |
| 720x480p            | 540p / 1080i | 480 lines become 512 (16 every 15 input lines, interpolated), centered. **4:3**: all 720 pixels, 2 clocks each (1440 wide); **Wide** (anamorphic 16:9): the ADC samples at 2.5x, 1800 pixels. 1080i: both fields show the same lines |
| 640x480p            | 540p / 1080i | as 720x480p, 640 pixels: **4:3** 2 clocks each (1280 wide), **Wide** 3 clocks each (1920 wide) |

* Every output pixel lasts a whole number of output clocks, and the output
  clock is generated at an exact ratio of the input clock, so horizontal
  scrolling stays smooth (480p / 640x480: every pixel is the average of
  exactly 2 ADC samples, i.e. one source pixel).
* Any other input (50Hz sources included) shows the test pattern with
  "Unsupported".


Installation
------------

1. Copy `ossc_hd_vX.Y.bin` to a micro SD card.
2. In the **official** firmware, install it with the firmware update function
   of the settings menu. The image carries the secondary-firmware key, so it
   is written to the secondary slot only (it replaces the OSSC Downscaler if
   that was installed).
3. Start it with **Launch 2nd FW**. The LCD shows `HD vX.Y 540p/i` and the
   OSSC outputs the test pattern until a supported input is found.
4. Press button 3 on the remote control (RGBHV) to switch to the correct input

Buttons on the OSSC: the 1st one selects the next input (as in the official
firmware); the 2nd one switches between 540p and 1080i (short press) and
between 4:3 and Wide for 480p / 640x480 (held 1 second).

To return to the official firmware, power cycle the OSSC.


Menu
----

| Item | Values | |
|------|--------|---|
| Output mode | 540p, 1080i | |
| CSync on HSYNC | Off (H+V), XOR, AND, Serrated | Off for HDMI displays and HDMI→component converters (the HDMI standard has separate syncs). Composite sync types are for RGBS monitors through an HDMI→VGA DAC; Serrated has equalizing pulses and serrated vsync |
| HDMI range | Full (0-255), Lim. (16-235), Lim. (16-255) | RGB levels sent on HDMI |
| 480p input | Auto, 720x480, 640x480 | 720x480 and 640x480 have the same line rate and are told apart by the hsync width; select the format manually if a source is misdetected |
| 480p format | 4:3, Wide 16:9 | 720x480 / 640x480: 4:3 pillarboxed, or anamorphic 16:9 |
| Flicker filter | Off, On | 1080p → 1080i: each field line averages 2 input lines (field B one line lower) instead of taking one |
| Line blending | On, Off | 1080p → 540p: the 2 input lines averaged (Off: the first one); 720p: weighted lines (Off: the nearest lines) |
| Line phase | 0–3 | 1080p / 720p: shifts the input line groups by whole lines, to align them with integer-scaled content |
| H. position | ±40 | moves the picture horizontally |
| V. position | ±12 | moves the picture vertically |
| Diagnostics | | `Rnnn`: output resyncs since boot (should stay constant while a mode runs); `Paaaa/bbbb`: measured / expected input frame period (low bits; must be equal) |
| Test pattern | Gray, Grid, Monoscope, RGB | design of the test pattern shown at start, with no sync or an unsupported input (select an unused input to see it). **Grid**: 16x9 cells with centre dots; **Monoscope**: circles, color bars, sharpness gratings, gray steps; **RGB**: red, green, blue and white ramps and steps. Not saved: Gray at start |

Settings are saved with **Settings opt. → Save settings** (stored in this
firmware's own sector, independent of the official profiles). The OSSC
Downscaler uses the same sector with a different format: settings are not
carried over when switching between the two, each starts from its defaults.


Known issues and limitations
----------------------------

* 50Hz sources are not supported.
* 540p over HDMI is not a standard format: flat-panel TVs may reject it or
  show stale lines at the bottom / right edge. Use 1080i with them.


Building
--------

The firmware is built by GitHub Actions (`.github/workflows/build.yml`) with
Quartus Lite and the RISC-V toolchain; the output is `ossc_hd_vX.Y.bin`, with
the version taken from `DS_VERSION` in
`software/sys_controller/inc/firmware.h`. See [README_OSSC.md](README_OSSC.md)
for the original OSSC build instructions.


Credits and license
-------------------

* Based on the [Open Source Scan Converter](https://github.com/marqs85/ossc)
  by Markus Hiienkari and contributors (see [CREDITS](CREDITS)).
* License: GPLv3, same as the OSSC project (see [LICENSE](LICENSE)).
