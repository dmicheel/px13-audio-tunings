# PX13 speaker EQ

Optional 20-band speaker EQ for the ASUS ProArt PX13, as a WirePlumber
software-DSP replacement for the raw speaker sink. Needs a working speaker
sink (from [px13-audio-fix](https://github.com/dmicheel/px13-audio-fix) or
a fixed upstream kernel) and WirePlumber 0.5+. Installs `lsp-plugins-lv2`
automatically if needed.

```sh
bash install.sh              # install + make default
bash install.sh --uninstall  # remove; restore the raw sink and its old volume
```

Runs as a regular user; sudo is only used to install `lsp-plugins-lv2`.

## What it does

`tunings/px13/` shapes the internal speakers with a fitted 20-band tonal
correction (~+2 dB bass/mids rolling off above 9 kHz). WirePlumber replaces
the raw ALSA speaker sink with one logical `Built-in Speaker (Tuned)` output,
so the desktop does not show a duplicate raw output. The hardware sink stays
at 100% behind the filter (set through the card's Speaker route with
`configs/px13-speaker-route.lua`, since the hidden sink can't be reached with
`pactl`); the tuned output volume controls the final level.
The audio still reaches the AMD ALSA sink because that is the hardware path to
the amplifiers.

- **Source:** 20 reference band targets at 48 kHz, see
  `tunings/px13/eq_targets.txt`.
- **Content:** the targets fitted as 20 peaking biquads/channel at Q=0.8
  (dense RMS error 0.08 dB vs target, table in `tunings/px13/tuning.conf`),
  plus a brickwall limiter at −1 dBFS. Headphones are untouched.
- **Out of scope, by design:** dialog enhancement, surround virtualization,
  dynamic compression, extra loudness stages.
- **Metadata:** `tunings/px13/tuning.conf` also carries `description` and
  `sink_pattern` lines for downstream auto-install tooling; nothing in this
  repo reads them.

A/B against the raw speakers by uninstalling the tuning. Before making the
tuned sink default, the installer plays a quiet 440 Hz probe into it, records
the filter output and checks that the gain matches the EQ curve. If the
replacement fails to start or the probe fails, it restores the previous
configuration and volume. Small EQ tweaks can be made in `filter-chain.conf`
(re-run `install.sh` afterwards, it installs a copy); the targets live
in `tunings/px13/eq_targets.txt`.
Status: math-verified only — not yet confirmed with a mic measurement.

Installs made from `px13-audio-fix` (before the split) are picked up: the
installer recognises their config files and moves their saved speaker volume
from `~/.local/state/px13-audio-fix/` to `~/.local/state/px13-audio-tunings/`.

## Layout

```text
install.sh            WirePlumber speaker EQ (--uninstall supported)
configs/              WirePlumber software-DSP rule + route script
tunings/px13/         filter-chain.conf, tuning.conf, eq_targets.txt
```

## License

Guide, script and WirePlumber configs: CC0. Tuning data (`tunings/`) is not
covered by any license grant in this repository. Details in `LICENSE`.

## AI notice

Parts of this repository (scripts, configs, and this README) were written
or revised with assistance from an AI coding tool, validated by hands-on
testing on the PX13. Review before use on other hardware.
