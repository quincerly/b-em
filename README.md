# Overview

This README is specific to the "pico" fork of b-em. See [here](README_b-em.md) for the original b-em readme.

The sole reason for this project's existence was to get a BBC B/Master 128 emulator running on the Raspberry Pi Pico. It supports
building on other platforms (particularly Pi) however that was not the initial intent, and the Pi version may be missing features you would like (since it has the same feature set as the Pico version). This may be addressed in the future, but is not high on my priority list at least, since most games can already be played.

NOTE: AS OF RELEASE THIS IS IN A "WORKS-ON-MY-MACHINES" STATE... SO EXPECT BUILD ISSUES!
# Running

## Keyboard Layout

Keys are layed out muchly the same as on a BBC Micro Keyboard; i.e. SHIFT-2 is `"`, SHIFT-' (next to Enter) `s '*`.

Function keys are one off; i.e. F1 = `F0`, F2 = `F1` etc.

F12 = `BREAK`

### Menu (for "Pico" versions i.e. RP2040, Pi etc)

F11, F15, LEFT_GUI, RIGHT_GUI = show/hide emulator menu.

When the menu is shown, use up/down arrows to move from option to option and left/right arrows to change selection. 
Some options take immediate effect, others will flash meaning there is a change pending. Hit Enter to confirm, or Escape to cancel the pending change. (note you cannot use up/down arrows until you Enter/Escape if you have a pending change). Note Escape from the menu level hides the menu.

## RP2040 Version

To support video output you will need a VGA breakout board for the RP2040, e.g. the [Pimoroni Pico VGA Demo Base](https://shop.pimoroni.com/products/pimoroni-pico-vga-demo-base).

Note that all RP2040 versions overclock the RP2040 and up the voltage to 1.25v, so use at your own risk!

Note there are new (as of March 7th 2021) variants which use 1080p 50Hz as the video mode, which will hopefully be recofgnized by more monitors than 1280x1024 50Hz

All discs are embedded within the binary; see [Embedding Discs](#Embedding-Discs) for instructions.

By default the emulator does not use USB host mode which would allow connecting a keyboard. Instead you should use
[sdl_event_forwarder](https://github.com/kilograham/sdl_event_forwarder) to send events from your host PC over UART.
The UART RX pin for receiving data is 21 by default.

Note (Feb 28 2021) , I just tried, and host mode USB is currently panic-ing, so that is not an option atm. (actually since then i have heard people having success, though it will interrupt your boo-beep and cause some screen corruption perhaps)

## Pi (or Linux) Version

Discs are still embedded within the binary; see [Embedding Discs](#Embedding-Discs) for instructions, however you can also
pass a disc via command line via `-disc path/to/disc`. Only .SSD and .DSD are supported.

# Building

## RP2040 Version

This should work on Linux and Mac (and possibly Windows; I haven't tried)

You need the Pico SDK: https://github.com/raspberrypi/pico-sdk
You need the Pico Extras: https://github.com/raspberrypi/pico-extras (put it next to your pico-sdk directory)

You also need to be setup to build with the Pico SDK (i.e. arm compiler etc).

To build:
```
mkdir pico_build
cd pico_build
cmake -DPICO_SDK_PATH=path/to/pico-sdk -DPICO_BOARD=vgaboard ..
make -j4
```

Note you might need `-DPICO_TOOLCHAIN_PATH=path/to/arm-gcc-install` also.

Outputs are `src/pico/beeb` and `src/pico/master` along with higher clock rate versions (which may or may not work for you)
`src/pico/beeb360` and `src/pico/master360`

You would add `-DUSE_USB_KEYBOARD` to use USB keyboard, but as I say this is currently broken. Note that I will probably make a micro host USB hid library anyway, since TinyUSB tends to sleep at awkard times currently anyway which can interfere with video.

## Pi Version

Right now only 32-bit OS is (probably) suitable for anything other than Pi 5; i say probably, becaus I haven't tried it recently, and a lot of the issues were with the graphics driver. The other issue is that the ARM thumb assembly (shared with the Pico version) doesn't compile for 64 bit, so you end up using the slower C version, but this shouldn't matter too much on anything except the very old Pi models.

For most predictable results, You should be on the latest Raspberry Pi OS with latest updates. 

As of right now, there seems to be some tearing under X, so Wayland is better. With previous Raspberry Pi OS versions, you wanted to disable desktop compositing (check previous versions of this README for suggestions).

You need the Pico SDK: https://github.com/raspberrypi/pico-sdk as this is nominally still a "Pico" build
You need the Pico Extras: https://github.com/raspberrypi/pico-extras (put it next to your pico-sdk directory)

You need the following packages installed:
```
sudo apt install build-essential cmake libdrm-dev libx11-xcb-dev libxcb-dri3-dev libepoxy-dev ruby libasound2-dev
```

To build:
```
mkdir pi_build
cd pi_build
cmake -DPICO_SDK_PATH=path/to/pico-sdk -DPI_BUILD=1 -DPICO_PLATFORM=host -DDRM_PRIME=1 -DX_GUI=1 ..
make -j4
```

Outputs are `src/pico/xbeeb` and `src/pico/xmaster`.

You can omit DRM_PRIME to build without that, but it isn't gonna help your frame rate.

Note the dependency on the Pico SDK could theoretically be removed, but it wasn't high on my priority list as it would have been 
a bunch of work making a whole separate abstraction for no benefit to my cause!

## Pi version on other Linuxes

You can run the Pi Build on other desktop Linuxes, omitting `-DDRM_PRIME=1` if your driver doesn't support DRM_PRIME/DRI3. It will use the slower C CPU emulation, but that shouldn't be a problem on any recent desktop.

## Regular B-em versions

This should work on Linux and macOS (and possibly Windows; I haven't tried)

You need allegro-dev installed. This is quite a hurdle on Raspberry Pi OS. If you have that, you can just do

```c
mkdir build
cd build
cmake ..
make -j4
```

You get:
* *b-em* - regular b-em
* *b-em-reduced* - b-em with similar stuff cut out as for the _Pico_ style build
* *b-em-reduced-thumb-cpu* - same as `b-em-reduced` but using the CPU from the src/thumb-cpu directory (mostly for comparative testing)

## "Host" Pico Build

This will compile the Pico version for `PICO_PLATFORM=host` using the `pico-host-sdl` project to simulate Pico audio and video output using SDL2

This should work on Linux and Mac, and on a Pi 4... it is gonna be a bit slow on older Pis (remember we're simulating bits of the the RP2040 SDK emulating a Beeb).


You need the Pico SDK: https://github.com/raspberrypi/pico-sdk
You need the Pico Extras: https://github.com/raspberrypi/pico-extras (put it next to your pico-sdk directory)
You need the Pico Host SDL: https://github.com/raspberrypi/pico-host-sdl (put it next to your pico-sdk directory)

You also need this on Linux/macOS (note use `brew install` for macOS)
```
sudo apt install libsdl2-dev libsdl2-image-dev
```

To build:
```
mkdir host_build
cd host_build
cmake -DPICO_SDK_PATH=path/to/pico-sdk -DPICO_PLATFORM=host -DPICO_SDK_PRE_LIST_DIRS=path/to/pico-host-sdl ..
make -j4
```

Outputs are `src/pico/beeb` and `src/pico/master`, however these use 1280x1024x50 which is a non-standard mode (it worked fine on all my monitors!)

Additionally now there are `src/pico/beeb_1080p` and `src/pico/master_1080p`.. these
require a slightly higher overclock (297Mhz vs 270Mhz), but most if not all RP2040 should still be able to do this I believe.

Finally there are some 360Mhz versions, but these which use the 1280x1024x50 mode again, are perhaps less interesting now `src/pico/beeb360` and `src/pico/master360`

## Embedding Discs

The build will embed discs based on the contents of [beeb_discs.txt](src/pico/discs/beeb_discs.txt) and [master_discs.txt](src/pico/discs/master_discs.txt). If you look at those files, you'll see:

* Some terse documentation
* That they each contain some default images
* That by default they are set up to use the contents of `beeb_discs_user.txt` and `master_discs_user.txt` from the same directory if those are present. This makes it easy for you to setup your own discs without changing the files checked into git.

If you want further control you can also set the CMake variable `OVERRIDE_DISC_FILE_<exe>` to the path of the discs file for that exe; e.g. for `xbeeb` it woudl be `OVERRIDE_DISC_FILE_xbeeb` (note i haven't actually tested this so lmk!)

Note: Only .SSD and .DSD are supported.

## Technical Details

See [here](src/pico/TECHNICAL.md) for technical details of the Pico port.

## Future Plans

* Fix cycle accuracy issues... this wasn't my highest priority; and having someone's help who knows the details more would likely help.

### RP2040 version

* USB keyboard etc. support
* I'd like to drive a real monitor using the beeb CRTC timings (probably would have done this if i had a monitor to test with - anybody? :-))
* Use a second pico as a TUBE/SPI adapter (either direction)
* Clearly serial, parallel and ADC should function correctly with pin outs! (perhaps with a GPIO extender for parallel)
* Tape support (clearly need to be able to hook up MP3 player or real tape deck!)
* Other real hardware (suggestions welcome)
