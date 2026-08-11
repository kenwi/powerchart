# powerchart

Log your laptop's **total system power draw**, battery charge % and battery
status, then chart it — in your terminal or in an interactive window.

The total draw is read from the battery's `power_now` sysfs entry (microwatts →
watts), so it reflects the whole machine: CPU, GPU, screen, Wi-Fi, everything.

## Features

- Lightweight bash sampler with a systemd user service (5s sampling by default)
- Tab-separated log with timestamp, watts, charge %, battery status
- Terminal chart with Unicode bars — no extra dependencies
- Interactive matplotlib window to probe exact values at a given time (`--show`)
- Optional PNG export
- No root required; everything runs in userspace

## Requirements

- Linux with a battery under `/sys/class/power_supply/` and a `power_now` entry
- `bash`, `date`, `awk`
- Python 3 (only for `powerchart --show` / `--png`; terminal chart is awk-free)
- matplotlib (optional, for `--show` / `--png`)
- systemd user session (optional — sampling can also run without it)

## Install

Clone and run the installer:

```bash
git clone https://github.com/you/powerchart.git
cd powerchart
./install.sh                 # default 5s sampling
./install.sh --interval 10   # sample every 10 seconds
```

This installs:

```
~/.local/bin/powerlog
~/.local/bin/powerchart
~/.config/systemd/user/powerlog.service   (enabled + started)
```

Log data accumulates at `~/.local/state/powerlog/power.log`.

### Manual install

```bash
install -m 0755 bin/powerlog    ~/.local/bin/powerlog
install -m 0755 bin/powerchart  ~/.local/bin/powerchart
mkdir -p ~/.config/systemd/user
cp systemd/powerlog.service     ~/.config/systemd/user/
systemctl --user daemon-reload
systemctl --user enable --now powerlog
```

## Usage

### Service control

```bash
systemctl --user status powerlog
systemctl --user restart powerlog
systemctl --user stop powerlog
systemctl --user disable --now powerlog
```

Log a single sample manually at any time:

```bash
powerlog --once
```

### Charting

```bash
powerchart                 # last 60 minutes
powerchart --since 15m     # last 15 minutes
powerchart --all           # entire log
powerchart --charge        # also show a charge % sparkline
powerchart --show          # interactive window; hover to probe exact values
powerchart --png           # render PNG to /tmp/powerchart.png and open it
powerchart --png ~/p.png   # PNG to a custom path
powerchart --help          # all options
```

Terminal output example:

```
Power draw — 1:00:00  (15 samples)
now 12.94W  ▁▁▃▃▄▄▆▆▆▇▇█▆▆▃  peak 13.80W / avg 13.22W / min 12.52W
     20:25:01                                                20:26:10
Charge %   89  ███████████▁▁▁▁  peak 90% / min 89%
Status: Discharging
```

## Log format

Tab-separated, one line per sample:

```
#timestamp	power_w	charge_pct	status
2026-08-11T20:25:01	12.52	90	Discharging
```

## Customization

| Variable | Default | Effect |
| --- | --- | --- |
| `POWERLOG_INTERVAL` | `5` | Seconds between samples |
| `POWERLOG_FILE` | `~/.local/state/powerlog/power.log` | Log file path |

Change the service's interval by editing the `--interval` value in
`~/.config/systemd/user/powerlog.service`, then:

```bash
systemctl --user daemon-reload && systemctl --user restart powerlog
```

### Keybinding (Hyprland example)

```conf
bindd = SUPER, B, Power chart (PNG), exec, /home/you/.local/bin/powerchart --png
```

## Uninstall

```bash
./install.sh --uninstall
```

This disables and removes the service, the scripts, and the log data.

## Notes

- `power_now` only reports while on battery; while charging, charge % still
  updates and status shows `Charging` / `Full`.
- The log grows ~17 KB/day at 5-second sampling (~43 KB/day at 2s).
