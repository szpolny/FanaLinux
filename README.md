# FanaLinux (Fanatec Linux Suite)

[![Linux Compatible](https://img.shields.io/badge/Platform-Linux%20(Fedora%20%2F%20Arch%20%2F%20Ubuntu)-blue.svg)]()
[![License](https://img.shields.io/badge/License-GPLv3-green.svg)]()
[![Hardware](https://img.shields.io/badge/Hardware-Fanatec%20CSL%20DD%20%2F%20Formula%20V2-orange.svg)]()

**FanaLinux** is a native, modern Linux GUI control suite providing comprehensive hardware tuning, profile switching, and dynamic telemetry feedback for Fanatec wheelbases and steering wheels.

Built using **GTK4** and **Libadwaita** on top of a user-space **`hidraw` / `evdev`** driver engine, it eliminates the need for proprietary Windows software or Wine wrappers to configure and tune Fanatec sim racing hardware.

<p align="center">
  <img src="pictures/overview.png" alt="FanaLinux Overview — CSL DD online" width="860" />
</p>

---

## Hardware support

| Base | Status |
|---|:---:|
| CSL DD / GT DD Pro | ![Validated](https://img.shields.io/badge/Validated-brightgreen) |
| Podium DD1/DD2, ClubSport DD/DD+ | ![Expected](https://img.shields.io/badge/Expected-yellow) |
| CSL Elite / ClubSport V2.5 (belt) | ![Unsupported](https://img.shields.io/badge/Unsupported-red) |

**Validated** — every tuning register confirmed on-wheel (CSL DD 5Nm + Formula V2, base 1.9.2.3 / motor 1.1.2.2). \
**Expected** — same protocol family and official parameter spec, encodings not yet verified on hardware. \
**Unsupported** — belt-driven bases are not supported; direct-drive bases only.

## Quick start

```bash
git clone https://github.com/szpolny/FanaLinux.git
cd FanaLinux
./install.sh
fanalinux
```

Flow: **Tuning** → adjust → **Apply to Wheelbase**. \
**Remember to long-press the Tuning button for the full Advanced menu** (all 14 parameters). \
Game force feedback needs the [hid-fanatecff](https://github.com/gotzl/hid-fanatecff) kernel module — You can install it from inside FanaLinux.


## Features

| Tab | What it does |
|---|---|
| Overview | Hardware status, kernel FFB driver install/update + version, live inputs, about and version |
| Tuning | Full parameter set (SEN/FF/FFS/NDP/NFR/NIN/INT/FEI/FOR/SPR/DPR/SHO/BLI/BRF/FUL), slots 1–5, disk presets with exact numeric entry + SEN quick-chips |
| Pedals | Software brake force (1–100%) for standalone USB ClubSport V3 / V3 Inverted, virtual controller, live physical/game brake meters |
| Profiles | Per-game auto-switching via `/proc` scan (Steam/Proton/Wine), auto telemetry start, fallback preset |
| Dyn FFB | Speed-sensitive + reverse-safety damping overlays |
| Vibration | ABS/engine/rev/suspension/TC/lock/spin/under-oversteer haptics with preview |
| Display | RevLEDs, flag LEDs, OLED gear/speed |
| Calibration | Center calibration, motor self-test, emergency cutoff |

**Telemetry in:** AC, BeamNG OutGauge, DiRT/WRC, F1, Forza (FM7 + FH/FM23), AMS2, SCS shm. \
**Out:** live RevLEDs, OLED gear/speed, ABS/TC flags, haptics, damping.

## Known limitations

- No read-back: The app can't display current on-wheel values — verification is eyes-on-OLED.
- Every Apply writes all known registers; DRI/ACP reset to 0/OFF. If you use Drift mode, re-set it on the wheel afterwards.
- SHO displays On/Off only; granularity unverified beyond that. FFS Linear confirmed by symmetry (Peak verified).
- BRF can't be validated without load-cell pedals.

## USB pedal brake force (this fork)

This fork adds the software brake-force feature from [BrakeTune](https://github.com/szpolny/braketune)
to [upstream FanaLinux](https://github.com/DavideDonadio/FanaLinux).
It works independently of a wheelbase connection and does not change the wheelbase's hardware BRF setting.

1. Connect ClubSport V3 / V3 Inverted pedals directly over USB (Fanatec `0eb7:183b`).
2. Quit BrakeTune or any other mapper that owns the pedals.
3. In **Pedals**, enable mapping and set brake force. At 100% the brake axis is unchanged;
   at 20%, one fifth of its range produces full braking. Throttle, clutch and buttons pass through.
4. Bind **FanaLinux Pedals** in your game and keep FanaLinux running (minimizing is fine).

Mapping is initially disabled. The enabled state and percentage are saved in
`~/.config/fanalinux/pedals.json`; enabled mapping resumes on the next launch.
The physical event device is grabbed while mapping runs to prevent duplicate input.
Disabling mapping, exiting FanaLinux or unplugging the pedals releases it and removes
the virtual controller. Enabled mapping retries automatically after reconnecting.
This fork does not add BrakeTune's tray/autostart behavior; closing the window quits mapping.
Pedals connected through a wheelbase are not mapped.

The installer offers to install `udev/70-fanalinux-pedals.rules` and load `uinput` at boot.
For a manual install:

```sh
sudo install -m 644 udev/70-fanalinux-pedals.rules /etc/udev/rules.d/
sudo install -m 644 udev/fanalinux-pedals.conf /etc/modules-load.d/
sudo modprobe uinput
sudo udevadm control --reload-rules
sudo udevadm trigger --subsystem-match=input
sudo udevadm trigger --subsystem-match=misc
```

Replug the pedals after changing permissions. On NixOS, declare these rules with
`services.udev.extraRules` and add `"uinput"` to `boot.kernelModules` instead.
Do not run the application as root. The Pedals status row reports mapper errors.

Run tests with `python3 -m unittest discover -s tests` (GTK tests need a display,
or `xvfb-run -a python3 -m unittest discover -s tests`).
To also exercise real kernel input/output with synthetic devices, run
`FANALINUX_UINPUT_TESTS=1 python3 -m unittest discover -s tests -p test_pedals.py`.
This requires access to `uinput` and the created event nodes; it never opens real pedals.

## Firmware updates

Flashing firmware needs [official software](https://www.fanatec.com/us/en/s/download-apps-driver). \
FanaLinux itself never flashes — by design.

## Legal

**FanaLinux is an independent community project and is not an official Fanatec product.** \
It is not affiliated with, endorsed by, or sponsored by Endor AG, Fanatec, Corsair, or their affiliates.

Protocol behavior is established through device testing and USB analysis for interoperability. \
No proprietary code, binaries, firmware, or assets are included. \
Fanatec product names identify compatible hardware only; all trademarks belong to their owners.

