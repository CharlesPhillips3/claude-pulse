# Claude Pulse

A local dashboard for Anthropic and Claude that runs on a Raspberry Pi 5 (or any Linux machine with Python 3.9+). It uses only the Python standard library, so there's nothing to `pip install`.

**What it shows**

- **ANT stock.** Live quote, the change today and against the $83 IPO price, the day range, volume and an estimated market cap. The chart covers 1 day, 4 days or every day since the IPO, with pre-market and after-hours shaded. It also lists Nasdaq, NVDA, GOOGL, AMZN and MSFT for context.
- **Service status.** Each component on status.claude.com, live incident updates, and scheduled maintenance with countdowns.
- **Incident timeline.** A 4-day outage bar and uptime % for each component, built from incident reports plus the Pi's own checks every 2 minutes.
- **Model speed and load.** Time to first token and tokens per second for Haiku, Sonnet, Opus and Fable, compared with each model's usual speed and labeled Faster than usual, Normal, Busy or Very busy. Includes 4-day charts and a "busiest hours" heatmap. *Needs an API key.*
- **Your usage and resets.** Countdowns for your 5-hour session and weekly limits, plus your API rate-limit headroom.
- **Release radar.** Flags a new Claude model the moment it appears in Anthropic's model list, and highlights headlines about releases.
- **News.** Headlines from the last week, filterable by Releases, Stock and Outages.

## Install (one line)

Open a terminal on the Pi and paste:

```bash
curl -fsSL https://raw.githubusercontent.com/CharlesPhillips3/claude-pulse/main/get.sh | bash
```

Run the same line again any time to **update**. Your settings and history are kept.

Or install from a clone:

```bash
git clone https://github.com/CharlesPhillips3/claude-pulse.git && cd claude-pulse && ./install.sh
```

The installer:

- copies the app to `~/claude-pulse`
- sets up a background service (`claude-pulse`) that starts on boot
- puts a **Claude Pulse icon on your desktop** and in the app menu (Internet section)
- optionally asks for an Anthropic API key

Then double-click the desktop icon, or browse to `http://localhost:8787`. Other devices on your network can use `http://<pi-ip>:8787`.

## Turning on model speed probes (optional)

1. Create a key at console.anthropic.com and give it a small monthly spend limit.
2. Put it in `~/.config/claude-pulse/config.json` as `"anthropic_api_key": "sk-ant-..."`.
3. Run `sudo systemctl restart claude-pulse`.

Every 15 minutes the Pi sends each model a tiny "count to 60" request (about 100 output tokens) and times the reply. That costs roughly a cent a day, depending on which models you probe. To change that:

- Raise `probe_interval_min` to probe less often.
- Edit `probe_models` to change which models get probed.

The same probe reads the API's rate-limit headers, and an hourly call to the model list powers the release radar.

## Why plan usage is typed in

Claude doesn't offer an API for claude.ai plan usage. To fill it in:

1. Open **Settings → Usage** in Claude.
2. Press **Update** on the dashboard and type in the percentages and reset times.

The countdowns then run on their own. When the weekly reset passes, it rolls forward 7 days automatically. When the 5-hour session reset passes, the session shows as reset.

## Handy commands

```bash
systemctl status claude-pulse          # is it running?
journalctl -u claude-pulse -f          # live log
sudo systemctl restart claude-pulse    # after editing the config
./uninstall.sh                         # remove (add --purge to delete history + config)
```

To change the port, set `"port"` in the config and re-run `./install.sh`, so the desktop icon points to the new port.

## Data sources and limits

- **Quotes:** Yahoo Finance's public chart endpoint, with Stooq as a fallback. Quotes can be delayed and this isn't investment advice.
- **Market hours:** The app knows the 2026–2027 Nasdaq holidays and early closes.
- **Status:** status.claude.com's public Statuspage API, falling back to status.anthropic.com.
- **News:** Google News RSS.
- **History:** stored in `~/.local/share/claude-pulse/pulse.db` (SQLite) and kept for 45 days. The 4-day charts for speed and the Pi's own status checks fill in as the app runs. Stock history and incident reports are available right away.
