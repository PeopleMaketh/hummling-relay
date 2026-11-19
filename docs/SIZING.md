# Relay Helper sizing guide

Version 0.1, draft. **These are planning estimates, not measurements.** They come from the published guidance and benchmarks of open-source media forwarding servers (see the sources at the end) and from simple arithmetic. We will replace them with our own load-test results. Always test on your own hardware before you rely on a number.

## 1. In one page

A relay helper forwards video and voice. It does not mix or re-encode them, so the **network is the limit first, the processor second, memory last.**

| Tier | Good for | Machine | Network |
| --- | --- | --- | --- |
| Small | one room of up to about 25 people, or a few small rooms | 2 vCPU, 4 GB memory | 100 Mbps in and out, public IP |
| Team | up to about 100 people on calls at the same time | 4 vCPU, 8 GB memory | 500 Mbps to 1 Gbps, public IP |
| Large | up to about 500 people at the same time | 16 vCPU, 32 GB memory | 10 Gbps, or several helpers |
| More | above that | several helpers; the app picks the least busy one | per helper as above |

"vCPU" is one hardware thread. "People" means people in calls, counted across all rooms on that helper. Figures assume typical meetings (see section 3). A call where everyone shows video in a large gallery needs more than one where most people only listen.

## 2. How to size it yourself

Use these steps and numbers, then check them with a load test (section 7).

**Per person:**

- sends one voice stream, about 32 to 40 kbps,
- may send one video stream in up to three quality levels at once, about 1.5 Mbps, 0.5 Mbps and 0.15 Mbps (together about 2.2 Mbps),
- receives at most three voices (the loudest) and up to nine video pictures: one large, about 1.5 Mbps, and up to eight small, about 0.15 to 0.25 Mbps each.

**For a room of P people:**

- data coming in to the helper, about P times 2.3 Mbps when everyone has video on, less when cameras are off,
- data going out, about P times 3 Mbps (range 2.8 to 4 Mbps) when everyone has video on,
- forwarded streams, about P times 12 (three voices plus nine video pictures per person).

**Processor:** forwarding costs processor time in proportion to the number of forwarded streams. A rule of thumb from mediasoup's own documentation is that one processor core handles about 500 forwarded streams, depending on the CPU. So a 50-person video room (about 600 forwarded streams) needs about 1 to 2 cores of forwarding work. Plan **twice that** for headroom, the operating system, secure connections and your other programs.

**Memory:** about 20 to 50 MB per person is a safe planning figure, plus about 500 MB for the program and the system.

## 3. Planning table (one meeting, everyone with video)

| People in the room | Data out | Data in | vCPU | Memory |
| --- | --- | --- | --- | --- |
| 10 | about 30 Mbps | about 23 Mbps | 1 to 2 | 2 GB |
| 25 | about 75 Mbps | about 58 Mbps | 2 | 4 GB |
| 50 | about 150 Mbps | about 115 Mbps | 4 | 4 to 8 GB |
| 100 | about 300 Mbps | about 230 Mbps | 8 | 8 GB |
| 250 | about 750 Mbps | about 575 Mbps | 16 | 16 GB, 10 Gbps card |
| 500 | about 1.5 Gbps | about 1.15 Gbps | 32, or two helpers | 32 GB, 10 Gbps card |

**Other shapes of meeting:**

- **Voice only:** 100 people is about 12 Mbps out and under 5 Mbps in; one or two vCPU is enough.
- **Webinar (a few people speak, many watch):** the many viewers dominate. Each viewer receives about 1.5 Mbps for the speaker's video, so 500 viewers need about 750 Mbps out.
- **Screen sharing:** a shared screen is a high-quality video stream. Budget about 1.5 to 3 Mbps per viewer while someone shares.

## 4. The machine

**Operating system:** Linux on 64-bit x86 or ARM is recommended (Ubuntu LTS, Debian, Amazon Linux). Windows and macOS are fine for small installs and trials.

**Run it as a container** where you can (Docker, ECS, Kubernetes). Use host networking or a wide UDP port mapping; a narrow port mapping is a common cause of calls that connect but have no video.

**Network:**

- a **public IPv4 address** that people can reach directly, not behind a second router or carrier-grade NAT (many home and mobile lines are; those cannot host a helper),
- **TCP 443** open for secure connections and a fallback path,
- **UDP 3478** open for connection setup,
- a **UDP range for media**, for example 50000 to 60000, or a single configured port,
- **upload speed** at least as big as the "data out" in the table, with room to spare. Most home lines have far less upload than download. If your upload is 50 Mbps, plan a room of about 15 people, not 50.

**Tuning on Linux (recommended for Team and above):**

- raise the open-files limit to at least 65535,
- raise the network buffers (for example, `net.core.rmem_max` and `net.core.wmem_max` to about 25 MB),
- use the "performance" processor setting and turn off sleep,
- keep the clock in sync (NTP),
- do not put a heavy load generator or other busy programs on the same machine.

## 5. Cloud examples (not endorsements, check current prices)

| Tier | Example instance types |
| --- | --- |
| Small | AWS c6i.large or t3.large; Google Cloud c2-standard-4; 2 vCPU class on any provider |
| Team | AWS c6i.xlarge or c6i.2xlarge; Google Cloud c2-standard-8 |
| Large | AWS c6in.4xlarge or larger (network-optimised); Google Cloud c2-standard-16 |

**Network charges usually cost more than the machine.** At about 3 Mbps out per person, one person uses about 1.35 GB an hour. At a typical public cloud internet price of about $0.09 per GB, that is about $0.12 per person per hour, or about $6 an hour for a 50-person meeting. Prices vary a lot between providers and with volume. A machine in an office or a data centre with a flat-rate connection is often much cheaper for steady use.

## 6. Running more than one helper

- Add helpers in different places (cities, or cloud regions). The app tests each person's delay to each helper and picks a nearby, healthy one.
- At first, one room lives on one helper. A single room is therefore limited to what one helper can carry. Linking several helpers for one very large room is a later feature.
- Scale out when any of these stays high for several minutes: processor above 60 percent, network above 70 percent of the card, or lost packets above 1 percent.

## 7. How to measure your own limits

1. Put the helper on the machine you want to test.
2. Run the load generator on **separate hardware** with plenty of processor and network. A generator on the same machine gives wrong results.
3. Make it look like your real meetings: the same number of senders, the same number of watchers, the same quality levels.
4. **Raise the load slowly** (for example, five people every ten seconds) while watching processor, network, memory and lost packets.
5. The limit is the load just before lost packets rise or the processor stays above about 75 percent.
6. Write the result in the table below and open an issue to share it.

| Date | Machine | People per room | Rooms | Data out | CPU | Lost packets | Notes |
| --- | --- | --- | --- | --- | --- | --- | --- |
| (to be filled in by our tests) | | | | | | | |

## 8. Safety and privacy

- Calls are scrambled end to end between the people in them. A helper forwards scrambled media and **cannot read it**. It can see who connects, when, from which address and how much data flows.
- A helper accepts only short-lived passes issued by Hummling for its workspace. It refuses to forward to private network addresses.
- Run it as a normal user, not as the administrator. Keep only the ports above open. Install updates when the app tells you; updates are signed.
- Registration keys work once, for 30 minutes, for one helper, and are removed when you remove the helper.

## 9. Not covered yet

- Recording, live captions and other features that need to read the media. They are not possible on a helper while calls are scrambled end to end.
- Linking several helpers into one very large room.
- Our own measured numbers. Treat every figure here as a starting point.

## Sources

- LiveKit, self-hosting benchmarks (method, reference hardware and the advice to run the load generator on separate hardware): https://docs.livekit.io/transport/self-hosting/benchmark/
- mediasoup, scalability (about 500 forwarded streams per processor core as a rule of thumb): https://mediasoup.org/documentation/v3/scalability/
- Network price figures in section 5 are typical public list prices. They change often, so check your own provider.
