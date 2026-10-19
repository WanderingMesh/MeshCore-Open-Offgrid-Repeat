# MeshCore-Open-Offgrid-Repeat

Patches and prebuilt firmware that let MeshCore's off-grid (client repeat) mode run
anywhere in the **902-928 MHz US/Canada ISM band**, instead of only on the three fixed
frequencies upstream allows.

---

## Why

Stock MeshCore companion firmware only permits off-grid/client-repeat mode on
433.000, 869.495, or 918.000 MHz. Turning repeat on therefore drags the whole group
onto one of those frequencies, so **every other companion in the mesh has to be
reconfigured by hand**.

For a small hiking party that is a minor annoyance. For a deployed 40-50 person SAR
team with vehicle-mounted companions it is a genuine operational problem: reconfiguring
that many radios is slow, it is a reliable source of misconfiguration, and once
personnel are in the field the ability to troubleshoot a mistyped frequency is close
to zero.

This change lets you enable a companion repeater **on the frequency the team is
already using**, so nobody else changes anything.

## Read this before you flash

Upstream restricts this **on purpose**, and the reasoning is sound: client repeating
is actively harmful on large public meshes, where every companion re-transmitting
turns into the congestion problem MeshCore was designed to avoid. That is why the
allowed list is short and why it sits away from the common regional presets.

This build is intended for **isolated, self-contained meshes** — a group operating out
of radio range of any public infrastructure. If you enable repeat on a public mesh
frequency and leave it on, you are degrading that mesh for everyone else on it.

The band is still bounded at 902-928 MHz. The goal was to drop a restriction that has
no regulatory basis, not to make out-of-band transmission easy to select. **You remain
responsible for operating legally in your jurisdiction** (in the US, FCC Part 15.247
governs this band). Nothing here changes TX power, duty cycle, or any other RF
parameter.

## Client app support is the catch

The firmware is only half the story. Enabling off-grid repeat is a flag in
`CMD_SET_RADIO_PARAMS`, and the client app **pre-validates the frequency before it ever
transmits the frame**. If your app hardcodes the old three-frequency list, it will
refuse to save and the device is never consulted — so this firmware will appear to do
nothing at all.

| Client | Status |
| --- | --- |
| MeshCore Open, build 17+ (PRE-BETA-9.5.2 and later) | **Works.** Queries `CMD_GET_ALLOWED_REPEAT_FREQ` and validates against whatever the device reports. |
| MeshCore Open, build 16 (PRE-BETA-9.5.1) | **Does not work.** Hardcodes `{433000, 869000, 918000}` and returns before sending the frame. |
| Official MeshCore iOS/Android app | **Unverified.** The app is closed source. See below. |
| MishMesh on-device UI (Wio Tracker L1) | **Works, no phone needed.** |

Both MeshCore Open builds report marketing version `9.5.0` in-app. The **build number in
parentheses** is the only reliable way to tell them apart. A secondary tell is the
warning text: build 16 says "433, 869, or 918 MHz", build 17 says "433, 869.495, or 918 MHz".

### On the official app

`CMD_GET_ALLOWED_REPEAT_FREQ` (command 60) returns *ranges*, not a list of points, and
it was added to the firmware in February 2026 when no other client existed — so it was
plainly built for the official app to consume. That is suggestive but **not proof**.
Two separate things have to be true and neither has been confirmed:

1. The app validates against the reported ranges rather than a hardcoded set.
2. Its UI lets you *enter* an arbitrary frequency with repeat enabled, rather than only
   offering presets. If it derives presets from each range's lower bound, a single wide
   range collapses to one useless "902.000 MHz" entry.

If you test it, please open an issue with the result.

## Prebuilt firmware

In [`firmware/`](firmware/). Each board has a `.uf2` (bootloader drag-and-drop) and a
`.zip` (nRF Connect / adafruit-nrfutil DFU).

| Board | File prefix | Flash used |
| --- | --- | --- |
| RAK4631 | `RAK_4631_companion_radio_ble_offgrid-companion-v1.17.1-4-ge9412598` | 494112 / 712704 (69.3%) |
| RAK4631 (MeshCore v1.18.0) | `RAK_4631_companion_radio_ble_offgrid-v1.18.0-offgrid` | 462772 / 712704 (64.9%) |
| RAK3401 + RAK13302 (WisMesh 1W) | `RAK_3401_companion_radio_ble_offgrid-companion-v1.17.1-4-ge9412598` | 431824 / 712704 (60.6%) |
| Wio Tracker L1 (MishMesh) | `WioTrackerL1_companion_radio_ble_offgrid_mishmesh_mm-v1.5.0-offgrid2_mc-v1.17.1-ff6b8bdc` | 666676 / 708608 (94.1%) |

SHA-256:

```
e8829aa9c180b8c91a6775f963ad21fc1177df0cf34a380ec064caea7ae0eb1d  RAK_3401_companion_radio_ble_offgrid-companion-v1.17.1-4-ge9412598-e9412598.uf2
16d8271fe70b925aa30588141928b25c3e431cac236d47b564674a1c02f8ddb5  RAK_3401_companion_radio_ble_offgrid-companion-v1.17.1-4-ge9412598-e9412598.zip
2e933c4e4933337eafcc2e79dd8d8710c35b986644a274c5757366eb5e34684b  RAK_4631_companion_radio_ble_offgrid-companion-v1.17.1-4-ge9412598-e9412598.uf2
ea58cb1f514a11d25e3bbb34227285520256932491b4a24611f248d107380fd0  RAK_4631_companion_radio_ble_offgrid-companion-v1.17.1-4-ge9412598-e9412598.zip
96535983b761e3e61b504935a37b8444f137989358627d2065bb15ee2cc85a89  RAK_4631_companion_radio_ble_offgrid-v1.18.0-offgrid-81dbae59.uf2
cf7091b8e8f713894c439e0f1d25c73b5c578652b118848e1b9c4d0bffa8687e  RAK_4631_companion_radio_ble_offgrid-v1.18.0-offgrid-81dbae59.zip
53621aab076983ca380ef8c61fdf5620d82b5ed9378bd8c2a11b02806ba64ab6  WioTrackerL1_companion_radio_ble_offgrid_mishmesh_mm-v1.5.0-offgrid2_mc-v1.17.1-ff6b8bdc.uf2
89ee9b157e379d6bfa3a9f14a0f84eb6e95fdae9fd3cfe4edca4a1ce9b6fc154  WioTrackerL1_companion_radio_ble_offgrid_mishmesh_mm-v1.5.0-offgrid2_mc-v1.17.1-ff6b8bdc.zip
```

All of these are nRF52840 (UF2 family `0xADA52840`). The RAK boards start at `0x26000`
(SoftDevice S140 v6); the Wio Tracker L1 starts at `0x27000` (S140 v7).

### Flashing

Double-tap reset to mount the bootloader drive, then copy the `.uf2` onto it. The board
reboots itself when the copy finishes.

Board-specific notes:

- **RAK3401 + RAK13302** — the 1W front end (SKY66122-11 FEM) is driven by the stock
  `variants/rak3401` config. These builds inherit `SX126X_REGISTER_PATCH=1` and
  `LORA_TX_POWER=22` from upstream unchanged.
- **RAK3401 with no display** — fine. The stock companion UI is status-only anyway; it
  has no repeat toggle and no settings editor, so off-grid repeat on this board is
  driven entirely by the phone app.
- **Wio Tracker L1** — MishMesh's on-device Repeater menu gains an `On <freq>` row.
  If you don't see that row, the offgrid firmware is not actually flashed.

## What changed

One table, in `examples/companion_radio/MyMesh.cpp`. It feeds all three validation
paths (`isValidClientRepeatFreq()`, the `CMD_SET_RADIO_PARAMS` handler, and the
`CMD_GET_ALLOWED_REPEAT_FREQ` responder):

```c
static FreqRange repeat_freq_ranges[] = {
  #ifdef ALLOWED_REPEAT_FREQ_RANGE
  ALLOWED_REPEAT_FREQ_RANGE
  #elif defined(OFFGRID_ALLOW_US915_BAND)
  { 902000, 928000 }
  #else
  { 433000, 433000 },
  { 869495, 869495 },
  { 918000, 918000 }
  #endif
};
```

The change is gated behind `OFFGRID_ALLOW_US915_BAND`, so **every other build target
keeps the stock three-frequency restriction**. Only the `_offgrid` envs are affected.

Reporting one wide *range* rather than a longer list of points is deliberate: the
companion protocol already models this as `{lower, upper}` pairs, so a
range-aware client accepts whatever frequency the group is already on without
needing any app-side change.

### Why not `ALLOWED_REPEAT_FREQ_RANGE`?

Upstream has an `ALLOWED_REPEAT_FREQ_RANGE` hook that looks purpose-built for this, but
**it cannot carry a `{lower, upper}` pair from a build flag**. SCons splits `-D` values
on commas, so `-D ALLOWED_REPEAT_FREQ_RANGE="{902000,928000}"` arrives as two separate
defines; the last one wins and the table silently collapses to a single `928000..0`
range that matches no frequency at all — repeat mode becomes impossible on *every*
frequency.

This fails quietly. It compiles with nothing worse than a macro-redefinition warning.
All three injection methods split the same way (`build_flags`, `env.Append(CCFLAGS=...)`,
and `env.Append(CPPDEFINES=[(name, value)])`), which is very likely why nothing in the
upstream tree actually uses the hook. Hence the range lives in the source and the build
flag only selects it.

### MeshCore v1.18.0

The table, `isValidClientRepeatFreq()`, the `CMD_SET_RADIO_PARAMS` check and the
`CMD_GET_ALLOWED_REPEAT_FREQ` responder are all unchanged in `companion-v1.18.0`, so
[`0001`](patches/meshcore-v1.18.0/0001-offgrid-repeat-band.patch) is the same hunk at new
line offsets.

1.18.0 also adds a companion CLI. It is reachable from the official app's Command Line
screen (MeshCore App v1.50.0+, per the release notes), from contacts you grant the Remote
CLI permission, and from the USB serial rescue CLI. Its `set radio freq,bw,sf,cr` only
range-checks 150-2500 MHz and **never consults the repeat table**, and nothing re-checks
repeat at boot. So a repeating node can be retuned anywhere in 150-2500 MHz and keep
repeating there after the next reboot. Stock 1.18.0 has the same hole around its three fixed points.

[`0002`](patches/meshcore-v1.18.0/0002-guard-cli-and-boot-repeat-band.patch) closes it,
behind the same `OFFGRID_ALLOW_US915_BAND` flag:

- While repeat is on, a `set radio` outside the allowed range is refused with
  `Error, repeat on: freq outside repeat range`. Turn repeat off first if you really mean
  to leave the band.
- At boot, repeat is switched off if the saved frequency is outside the allowed range.
  That catches prefs written by firmware without the guard, such as stock 1.18.0.

`CMD_SET_RADIO_PARAMS` and `set radio` are the only two paths that change the frequency
in 1.18.0, so with both guards the band bound holds on all of them. Turning repeat *on*
is still done through the app's radio settings, since the CLI has no repeat command, so
the client support notes above apply unchanged.

### MishMesh additions

MishMesh has an on-device UI, which needs two extra things:

- A `repeat_freq_presets[]` array, kept **separate** from `repeat_freq_ranges[]`.
  The ranges table now describes what is *legal* (a band), and rendering a band as menu
  rows would collapse the picker to one entry at the band's lower edge.
- An **"On `<freq>` MHz"** row in `RepeaterApplet`, which enables repeat *without*
  retuning the radio. That is the whole point: the mesh keeps its existing
  freq/bw/sf/cr and nobody else reconfigures anything.

### MishMesh radio-settings revert fix (`offgrid2`)

The first Wio build (`mm-v1.5.0-offgrid`) inherited an upstream MishMesh bug that
made radio changes made *from the phone app* appear to revert, or get stuck on a
previous frequency. `offgrid2` fixes it. Symptoms were:

- Setting a frequency in the app, having it snap back to the previous value.
- The same behaviour whether off-grid repeat was on or off.
- A reboot letting the change stick, then the *new* value becoming the one it
  reverted to.

`RadioSettingsPanel` is a singleton that stays on the applet stack while the
screen sleeps, and `AppletHost` pops it on wake (`popToRoot` -> `onStop` ->
`onHide`). Its `onHide` committed the **entire** staged `RadioConfig` that had
been captured when the panel was opened, so a config changed over the companion
link in the meantime was silently overwritten — frequency, bandwidth, SF, CR and
TX power alike. `onFreqDone` also set the dirty flag unconditionally, so merely
opening the frequency keypad and confirming was enough to arm that overwrite.

The fix is three-part:

- Field-level dirty tracking. `onHide` now re-reads the live config and overlays
  **only the fields the user actually edited**, so an untouched field can never be
  clobbered.
- `stage*()` setters no-op when the value is unchanged, so confirming a keypad
  without editing leaves the panel clean.
- A new `onShow()` re-seeds the staged copy from live config, but only when
  nothing is pending, so returning from a sub-screen preserves an in-progress edit
  while a freshly revealed panel picks up external changes.

While fixing that, `repeat_saved_freq` handling moved out of `RepeaterApplet`
entirely. The picker only ever sees *staged* values, so writing that pref inline
could record a restore target the radio was never tuned to — and it wrote to flash
even if the user then backed out. `RadioSettingsPanel::onHide` now derives it from
the committed before/after state, and a deliberate retune clears it so the
Repeater menu's "Off" row cannot snap the radio back to a stale frequency.

Covered by 7 new tests in `test/test_mishmesh_radio/` and
`test/test_mishmesh_repeaterapplet/`; the suite is 964 tests and all pass.

Note that the first two parts fix **upstream MishMesh behaviour**, not the off-grid
change. Any MishMesh user who edits radio settings from the phone app while the
device screen is awake is exposed to the original bug.

## Building from source

Requires PlatformIO. Patches are verified to apply cleanly to the exact upstream commits
below and to reproduce these binaries.

### MeshCore v1.17.1 (RAK4631, RAK3401)

```sh
git clone https://github.com/meshcore-dev/MeshCore.git
cd MeshCore
git checkout e94125987ed87497e706a0b54d1e80c709343980   # companion-v1.17.1-4-ge9412598

git apply /path/to/patches/meshcore/0001-offgrid-repeat-band.patch
cp /path/to/patches/meshcore/platformio.local.ini .     # gitignored; adds the _offgrid envs

export FIRMWARE_VERSION=$(git describe --tags)
sh build.sh build-firmware RAK_4631_companion_radio_ble_offgrid
sh build.sh build-firmware RAK_3401_companion_radio_ble_offgrid
```

Artifacts land in `out/`.

> **`build.sh` runs `rm -rf out` on every invocation.** Copy each artifact somewhere
> safe before building the next target, or the second build will delete the first.

### MeshCore v1.18.0 (RAK4631)

```sh
git clone https://github.com/meshcore-dev/MeshCore.git
cd MeshCore
git checkout 81dbae592ffad40b4212bc0b37e82b38da08f4bc   # companion-v1.18.0

git apply /path/to/patches/meshcore-v1.18.0/0001-offgrid-repeat-band.patch
git apply /path/to/patches/meshcore-v1.18.0/0002-guard-cli-and-boot-repeat-band.patch
cp /path/to/patches/meshcore-v1.18.0/platformio.local.ini .   # gitignored; adds the _offgrid env

export FIRMWARE_VERSION=v1.18.0-offgrid
sh build.sh build-firmware RAK_4631_companion_radio_ble_offgrid
```

The device reports `FIRMWARE_VERSION` plus the `HEAD` hash, i.e. `v1.18.0-offgrid-81dbae59`,
so these units can be told apart from stock 1.18.0 in the app. Leave the patches
uncommitted so that hash stays the public upstream commit.

1.18.0 moved to GCC 14.2 (`toolchain-gccarmnoneeabi@1.140201.0`) and a newer RadioLib, so
the first build downloads new packages and can take 15 minutes or more.

The patches are board-agnostic, but only the RAK4631 has been built and checked on 1.18.0
so far. The RAK3401 and Wio builds remain on 1.17.1.

### MishMesh (Wio Tracker L1)

```sh
git clone https://github.com/burakcan/MeshCore-mishmesh.git
cd MeshCore-mishmesh
git checkout ff6b8bdc5928652bb70d6fef4dc4a4e8b7e072ce   # mishmesh-v1.5.0

git apply /path/to/patches/mishmesh/0001-offgrid-repeat-band-and-ui.patch
cp /path/to/patches/mishmesh/platformio.local.ini .

export MISHMESH_VERSION=v1.5.0-offgrid2
export FIRMWARE_VERSION=v1.17.1
sh build.sh build-firmware WioTrackerL1_companion_radio_ble_offgrid_mishmesh
```

> The env name **must** end in `_mishmesh`. `build.sh` branches on that suffix; a name
> like `..._mishmesh_offgrid` falls through to the generic MeshCore path, which demands
> `FIRMWARE_VERSION` and skips the `-DMISHMESH_VERSION` the About screen reads.

Run the unit tests with `pio test -e native` (964 tests, including coverage for the
`On <freq>` row and the radio-settings revert fix).

## Verifying a build

Two checks worth running, since the failure modes here are quiet.

**Band constants.** Build the stock env alongside the offgrid one and compare the raw
images. The three fixed literals should disappear and the band pair should appear:

```sh
arm-none-eabi-objcopy -O binary .pio/build/<env>/firmware.elf fw.bin
python3 -c "
import struct,sys
d=open('fw.bin','rb').read()
for v in (433000,869495,918000,902000,928000):
    print(v, d.count(struct.pack('<I',v)))"
```

Expect `0` for the first three and `1` each for 902000/928000.

Note that you **cannot** find a fixed frequency by scanning for a contiguous
`{lower, upper}` byte pair — the optimizer folds `f >= X && f <= X` into `f == X`, so
each fixed frequency leaves only a single literal in `.text`. Scan for individual
constants.

**UF2 header.** Check every block has valid magics (`0x0A324655`, `0x9E5D5157`,
`0x0AB16F30`), sequential indices, family id `0xADA52840`, and the right start address
for the board's SoftDevice.

**MeshCore v1.18.0 builds** need a slightly different band-constant check:

- Scan the `firmware.bin` inside each build's `.zip` after copying it out of `out/`. A
  second `build.sh` run (for the stock env, say) can clear `.pio/build` as well as `out/`.
- Count only 4-byte-aligned matches. GCC 14 leaves each fixed frequency in two literal
  pools, and there is also an unrelated, unaligned `918000` byte pattern inside
  `Adafruit_BusIO_Register::write`.
- The `0002` guard's error string should be in the offgrid image and absent from stock,
  which confirms the guard compiled in only behind the flag.

```sh
unzip -p RAK_4631_companion_radio_ble_offgrid-v1.18.0-offgrid-81dbae59.zip firmware.bin > fw.bin
python3 -c "
import struct
d=open('fw.bin','rb').read()
for v in (433000,869495,918000,902000,928000):
    p=struct.pack('<I',v)
    print(v, sum(d[i:i+4]==p for i in range(0,len(d)-3,4)))
print('guard', d.count(b'freq outside repeat range'))"
```

In the order printed, expect counts of `0 0 0 1 1` and `guard 1` for offgrid, and
`2 2 2 0 0` and `guard 0` for stock.

**On the device (v1.18.0).** From a client with a Command Line screen:

1. `ver` reports `v1.18.0-offgrid-81dbae59`.
2. `get radio` shows the current `freq,bw,sf,cr`. Reuse its bw/sf/cr below.
3. With repeat on, `set radio 869.525,<bw>,<sf>,<cr>` is refused with
   `Error, repeat on: freq outside repeat range`.
4. With repeat on, an in-band `set radio 911,<bw>,<sf>,<cr>` replies `OK - reboot to apply`,
   and after the reboot the node is still repeating, now on 911. Set your own frequency
   back the same way afterwards.

## Credits and license

All firmware source is from the MeshCore project and is **MIT licensed** — see
[`LICENSE-MeshCore.txt`](LICENSE-MeshCore.txt). The patches in this repository are
offered under the same terms.

- [MeshCore](https://github.com/meshcore-dev/MeshCore) — the upstream firmware.
- [MishMesh](https://github.com/burakcan/MeshCore-mishmesh) — the MeshCore fork adding a
  standalone on-device UI.
- [MeshCore Open](https://github.com/zjs81/meshcore-open) — the unofficial MIT-licensed
  Flutter client, and the first one to consume `CMD_GET_ALLOWED_REPEAT_FREQ`.

This repository is not affiliated with or endorsed by the MeshCore project. Please do
not file issues upstream about these builds.
