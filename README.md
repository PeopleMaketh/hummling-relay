# Hummling Relay Helper

A relay helper is a small program you install on a PC or a cloud machine and register with your Hummling workspace using a one-time key. Once it is registered, Hummling uses it to carry group calls, the way a conferencing service does: each person sends one stream to the helper, and the helper forwards the right streams to everyone else. Without any relay helper, calls fall back to direct person-to-person connections, which only work well for small calls.

**Status: planning. There is no program to download yet.** This repository holds the plan and the hardware sizing guide so you can prepare machines in advance.

## What is here

| File | What it is |
| --- | --- |
| [docs/SIZING.md](docs/SIZING.md) | The device specification guide: how much CPU, memory and network a relay helper needs for a given number of people, how to set up the machine, and how to measure your own limits. |

## How it will work (short)

1. In the Hummling app, an admin chooses **Settings, Relay helpers, Add a relay helper**, names it and gets a one-time key that works for 30 minutes.
2. On the machine, install the program (Windows, macOS, Linux) or run the container image, and paste the key.
3. The helper connects out to Hummling, proves the key, and Hummling tests its speed and that it can be reached from the internet.
4. When someone starts or joins a call, Hummling picks the nearest healthy helper and gives each person a short-lived pass for it. No pass, no relay.
5. Calls are scrambled end to end between the people in the call. A helper forwards scrambled media and cannot read it. It does see who connects, when, and how much data flows.

## What a relay helper does not do

- It does not store or record calls.
- It does not hold your chat messages. Those stay on your Hummling server.
- It does not serve other workspaces. A helper is registered to one workspace.

## Feedback

Numbers in the sizing guide are planning estimates until we publish our own load-test results. If you measure something different, please open an issue with your hardware and results.

## Licence

Apache License 2.0. See [LICENSE](LICENSE). Copyright 2026 People Maketh.
