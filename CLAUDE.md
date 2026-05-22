# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**roco-merchant-notifier** - a Python automation tool that monitors the "Traveling Merchant" (远行商人) in the game "Roco Kingdom World" (洛克王国世界). It fetches merchant inventory data, renders it as a styled image, and pushes notifications to mobile devices.

## Commands

### Run
```bash
python main.py
```

### Install Dependencies
```bash
pip install requests playwright jinja2
playwright install chromium --with-deps
```

No build step, no test suite, no linter configured.

## Required Environment Variables

| Variable | Required | Purpose |
|---|---|---|
| `ROCOM_API_KEY` | Yes | WeGame API access key |
| `IMGBB_KEY` | Yes | ImgBB image hosting API key |
| `FEISHU_WEBHOOK` | Yes | Feishu custom bot webhook URL |

## Architecture

The application is a single-file pipeline (`main.py`) with five sequential stages:

1. **Fetch** - GET request to `wegame.shallow.ink` API with API key header
2. **Process** - Filters products by Beijing time (UTC+8), calculates current round (1-4, each 4 hours starting 08:00), separates active vs. historical products
3. **Render** - Jinja2 injects processed data into `assets/yuanxing-shangren/index.html`, Playwright captures `.merchant-page` element as JPEG at 900px width
4. **Upload** - Posts rendered image to ImgBB, returns public URL
5. **Push** - Sends notifications via Bark (iOS) and NotifyMe (Android); skips unconfigured channels

### Key Design Decisions

- **Time handling**: All time logic uses Beijing timezone (UTC+8). Merchant rounds are 4-hour cycles: 08:00, 12:00, 16:00, 20:00.
- **Rendering**: Playwright renders a local HTML file with custom Chinese fonts (FZLant, DunDun, HYWenHei) bundled in `assets/yuanxing-shangren/ttf/`. The temp HTML is written to disk then deleted after screenshot.
- **History grouping**: Past rounds from today are grouped by time window, capped at 5 products per group, and shown in a separate "今日其他时段" section.

## Deployment

Runs via GitHub Actions (`.github/workflows/schedule.yml`). The cron schedule is currently commented out; triggering is done externally via `workflow_dispatch` (e.g. from cron-job.org) to avoid GitHub Actions scheduling delays.
