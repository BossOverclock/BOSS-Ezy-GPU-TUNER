# BossOverclock — BOSS-Ezy-GPU-TUNER

**v0.01 · "The Stability wins the game"**

A Windows overclocking tool for your GPU and CPU with a live graph for every reading that
matters — and the first temperature alarm system of its kind in a GPU tuner. It is free for
everyone, always.

Screenshots of the app as shipped: `docs/main-v0.01.png` (main window) and
`docs/monitor-v0.01.png` (the detached monitor panel).

## What this app does

You set the offsets, the app applies them, and the app keeps watching: every control has a
graph that records the real value over time, so you see what your card actually did under
load, not just what you asked it to do.

The readings it tracks:

- **Core clock, Xbar clock, Memory clock** — did the offset you asked for actually land, and
  did it hold through the workload?
- **CPU temperature, GPU hot spot temperature, Memory junction temperature** — the hot spot is
  where heat really piles up, and the memory junction is where the VRAM starts suffering. Both
  are hidden from most tools. The CPU number matters too: clocks are only as good as the
  platform feeding them.
- **Core voltage, power draw, GPU usage, VRAM usage** — the pressure the card is under.
- **Two temperature alarms you set yourself** — see the alarm section below.
- **Hover any graph**: a crosshair lands on the exact sample, showing the value and the exact
  time it happened. *♥♥♥ Was that dip at 21:16:30, right when I raised the
  offset?* — now you can answer that.
- **5 profile slots.** Save an aggressive set, save a stable set, switch between them, apply at
  startup. Measure, nudge, re-measure. That loop is tuning, and this app is built to make the
  loop short.

The honest answer to "how far can this card go?" is not a number from a benchmark. It is: the
highest clocks that stay true under your real workload, with the hot spot and junction under
control. Find that, keep it, and the game is won.

## The alarm system — the first of its kind in a GPU tuner

Most tuning tools show you a temperature and leave you to stare at it. BossOverclock ships the
alarm system for your GPU hot spot and your CPU, built into the tool from version 0.01:

- **GPU hot spot alarm** — default **95 °C** until you change it.
- **CPU temperature alarm** — default **95 °C** until you change it.
- Both thresholds are yours: type a number in Settings, it saves instantly, **0 = off**, clamp
  0–130 °C.
- When a sensor reaches your number: that panel turns **red**, its title gets a ⚠ ALARM
  marker, the dashed alarm line on the graph shows where your threshold sits, the status line
  tells you what to check, and an **audible alarm sounds** (a real WAV tone, not the silent
  Windows beep).
- When it drops back below your number, the panel returns to teal and the status line says the
  alarm cleared.
- The alarm fires **the moment it crosses**, not at the end of a run — you hear it while you
  can still act: lower the offset, raise the fan, back off the voltage. Heat is what steals
  stability, so you get to watch it *and hear it*.

That is the point. A tool that shows you heat tells you what happened. A tool that alarms you
tells you what is happening.

## v0.01 feature list

- **Core Clock / Xbar Clock / Memory Clock** sliders (−1000 to +1000 MHz; memory up to
  +3000 MHz)
- **Fan Speed** control (0 = auto)
- **Power Limit** in watts, with a percent mode toggle
- **GPU hot spot alarm + CPU temperature alarm**, both default 95 °C — red panel, status
  warning, audible sound, alarm line drawn on the graph
- Detachable **Hardware Monitor** window, docked on the right of the console at startup, zoom
  with Ctrl + wheel and ＋/−
- Panel order, top to bottom: **Core clock · GPU voltage · CPU temperature · GPU
  temperature · GPU hot spot · Memory temp (junction) · Xbar clock · Memory clock ·
  GPU usage · VRAM usage · Power draw** — the core clock leads, GPU voltage sits right
  under it so the two numbers you tune together are read side by side, then the four
  temperatures, then the rest
- Every panel shows current value, Min, Max and the time, in fixed gutters so the graph line
  never crosses the text
- Hover readout on every graph: crosshair, amber dot, value + timestamp box
- Live header line: **CPU, GPU temp, GPU hot spot, GPU mem temp** — one line, fully visible
- Live GPU info: name, driver version, architecture, core count, VRAM size, power limit, plus
  HWiNFO voltage readout to 3 decimals
- The app icon is the BossOverclock logo
- Profiles 1–5, Save / Apply / Reset, apply-at-startup, zoomable UI
- Compact, fully visible status line: after Apply it reports every control with its own result
  — `Applied: GPU→3200 MHz OK  Xbar +300 MHz ok  Mem +3000 MHz ok  Power 350 W ok` — and
  the status row grows so no part of the message is cut off
- Short footer line `v0.01 · BossOverclock · 2026 RealB · About` — the About popup
  carries the full legal text, license, credits and the donation addresses
- Window fits any screen: opens centred, never runs off the edge, zoom stops at the biggest size
  your display can show, scales down on small laptop panels, PerMonitorV2 DPI aware

The Advanced tab (voltage demands, SYS/Video/MSVDD ratios, OCP, voltage offsets) exists in the
app as a v0.02 feature and is not shipped in v0.01.

## How the control works

Three sources, each used for what it is good at:

1. **NVML** — ships with the NVIDIA driver, no redistribution needed. Clock locks, fan, power
   limit, utilisation and VRAM reads, GPU temperature, driver version, power-range limits.
2. **An engine CLI** — the deep voltage / ratio / boost controls. The app talks to a
   command-line engine binary named `bossengine.exe` (or `engine.exe`) placed **next to the app
   exe** (or in a parent folder). You supply your own engine binary from its author''s project
   and copy it as `bossengine.exe`; **it is not included here**, because redistribution is not
   allowed by its author. The app invokes it through its documented command line only.
3. **HWiNFO shared memory** — the readings NVML cannot give.

### Do I need HWiNFO?

Honest answer: **yes, keep it running.** HWiNFO is free, needs no account, and only publishes
sensor values into a shared memory section; it does nothing to your GPU.

What HWiNFO gives that nothing else does:

- **GPU hot spot temperature** — Nvidia''s API does not expose it at all
- **GPU memory junction temperature** — NVML sensor 2 returns −1 on the 621.35 driver
  (RTX 5070 Ti)
- **Measured core voltage** — the real volts, not the value you asked the driver to set

Reading map:

| Reading | Source | Fallback |
| --- | --- | --- |
| Core / memory clock, fan, power, usage, VRAM | NVML | — |
| GPU temperature | NVML | — |
| GPU hot spot, memory junction, measured voltage | HWiNFO shared memory | none, keep HWiNFO open |
| CPU package temp | HWiNFO `CPU (Tctl/Tdie)`, then `CPU Die (average)`, then a CCD Tdie — the
  value Ryzen Master shows | Windows thermal zone counter, panel labelled `CPU temp (Windows)` |

## Running the app

- **Windows 10 / 11, x64.**
- **.NET 8 Desktop Runtime** is required for the small app build in this repo
  (`https://dotnet.microsoft.com/download/dotnet/8.0`). Machines that already have .NET 9 or 10
  run it as is. The self-contained build, with the runtime inside (154 MB), is not hosted here
  — it is too large for GitHub — ask in Discord and we send it.
- **Administrator rights are required** — the driver writes need them.
- **HWiNFO running** for hot spot, memory junction and measured voltage.
- `bossengine.exe` next to the app exe for the deep voltage controls; without it the app still
  runs, the engine features report as unavailable.

## Config and startup

- Config: `%AppData%\BossOverclock\config.json` — 5 profile slots (all fields), startup tick,
  startup slot, hot spot alarm, CPU alarm.
- Apply-at-startup: the app writes a Windows Run key and launches with `-startup` at logon. It
  applies the chosen slot: clocks, fan, power limit. Measured voltage demands are not applied at
  boot, so the engine side stays a manual Apply.
- The app writes the Run key and reads shared memory; it has **no network access** — nothing is
  uploaded, no telemetry, no account.

## Testing

The app is driven by a scripted test harness on the real hardware: it pumps values into the
panels, runs the alarm logic, applies profiles, reads the config round-trip, and captures
screenshots of the real windows. Current state: **31 checks, 30 pass, 1 known limitation** —
NVML memory-junction temperature returns −1 on driver 621.35 (RTX 5070 Ti), which is why
HWiNFO is the primary source for that reading.

## What is in this repo

```
BOSS-Ezy-GPU-TUNER/
├── README.md              this file
├── LICENSE.txt           license, © 2026 RealB
├── .gitignore
├── app/
│   ├── BossOverclock.exe        the app, with the BossOverclock logo as its icon
│   ├── bridge/HwinfoCli.dll     HWiNFO shared-memory reader bridge
│   ├── bridge/ijwhost.dll       bridge host library
│   └── bossengine.exe           NOT in this repo — you supply it, named bossengine.exe
└── docs/
    ├── main-v0.01.png           main window as shipped
    └── monitor-v0.01.png        monitor panel as shipped
```

The source code is not published in v0.01 — this repo carries the runnable app, the license,
the docs and the notes. If you want to see the code, open the About popup in the app and ask in
Discord — the code is the work, and it goes public when it is ready to be used safely by
others.

Not in this repo, on purpose: `bossengine.exe` (its author bans redistribution) — you supply
it.

## The tool is free — for everyone, always

BossOverclock is free. No account, no key, no paywall, no limits, no ads. Everyone gets the
same build, the same features, the same price: nothing.

If you want to buy the author a coffee, or leave a small donation, that is much appreciated —
**with big smiles**. It keeps the work going between the long hours of testing, and it is
entirely optional. ETH and BTC addresses are in the app (♥ button, QR + copy address) and
in the About popup.

## Discord

Feature requests, bug reports, the roadmap, and the self-contained build —
**https://discord.gg/r3kqCpMh** — see you there.

## Roadmap

- **v0.02** — the Advanced tab, unhidden and polished: voltage demands, SYS/Video/MSVDD
  ratios, OCP, voltage offsets.
- Fan control refinement, alarm history, and a dark About popup.

## License and credits

© 2026 RealB. All rights reserved. See `LICENSE.txt`. You may not redistribute, decompile,
disassemble, or binary-analyze this software without written permission. Engine binary terms are
defined by its author. HWiNFO is a proprietary application, used only through its shared-memory
interface; hardware reading is credited to the MIT-licensed hwinfo-main library. NVML ships with
the NVIDIA driver. Use this tool at your own risk — overclocking raises heat and can shorten
hardware life.

Built by RealB. "The Stability wins the game." — v0.01, 2026.
