# Nexus Network Lab: Application + Transport Layer Visualizer

An interactive, single-file teaching tool that shows how a client and a server talk to each other, step by step. Each exchange is shown at two layers of the TCP/IP model:

- **Application layer:** HTTP, SMTP, DNS, TLS
- **Transport layer:** TCP segments and UDP datagrams

Everything runs in the browser. There is no backend and no network calls. The only external request is Google Fonts for typography, which falls back to system fonts if offline.

---

## What's new in v2 (transport layer)

v1 visualized the application layer only. Transport behavior was squashed into single log lines like `SYN → SYN-ACK → ACK`. v2 makes the transport layer a first-class part of every step.

| Area | v1 | v2 |
|---|---|---|
| Layers shown | Application only | Application **and** transport, stacked in each step |
| TCP handshake | One combined log line | Three separate steps: `SYN`, `SYN,ACK`, `ACK` |
| TCP teardown | One line, `FIN → ACK` | Three steps: `FIN,ACK`, `FIN,ACK`, `ACK` |
| Transport headers | Not shown | Ports, seq, ack, flags, window, payload size (TCP); ports, length, checksum (UDP) |
| Sequence numbers | None | Tracked per connection and advanced correctly (see below) |
| DNS | Labeled generically | Sent over **UDP** (port 53), with no handshake |
| Encapsulation | Not shown | A bar on each step shows `[ TCP header \| HTTP data ]` |
| Explanations | None | A short teaching note on steps where the transport behavior matters |
| Packet animation label | `HTTP` | `HTTP / TCP`, or `TCP SYN+ACK` for control segments |
| Layer toggle | n/a | Checkbox to hide transport details and get the v1-style app-only view |
| Large responses | Not modeled | Payloads over one segment are capped at MSS 1460 B, with a note that TCP splits the rest |
| Playback speed | 1.3 s per step | 1.6 s per step, to leave time to read the extra detail |
| Step counts | Fewer, handshake collapsed | See "Scenarios" below |

The page layout, theming, scenario tabs, and playback controls (Replay, Back, Play/Pause, Forward) are unchanged.

---

## Running it

Open `nexus-network-lab.html` in any modern browser. No build step or dependencies are needed.

1. Pick a scenario on the left (Web browsing, Sending mail, or Video streaming).
2. Adjust the inputs if you like (URL, email fields, or playback quality).
3. Press the action button. The exchange plays automatically, one step at a time.
4. Use **Pause / Back / Forward / Replay** to go through it at your own pace.
5. Use the **Show transport layer** checkbox to hide or show the transport details.
6. Use the 🌓 button in the header to switch between light and dark mode. The page follows your system setting on load.

---

## Scenarios

| Scenario | App protocol | Transport | Server port | Steps |
|---|---|---|---|---|
| Web browsing (`http://`) | DNS, HTTP | UDP for DNS, TCP for HTTP | 80 | 10 |
| Web browsing (`https://`) | DNS, TLS, HTTP | UDP for DNS, TCP for TLS and HTTP | 443 | 12 |
| Sending mail | SMTP | TCP | 25 | 17 |
| Video streaming | HTTP (DASH-style) | TCP | 443 | 12 |

**Web browsing.** A DNS query and answer go over UDP. A TCP handshake follows. With `https://` URLs, a TLS ClientHello/ServerHello exchange comes next. Then comes the HTTP GET and 200 response, then the TCP close. The URL you type drives the host, path, and port.

**Sending mail.** A TCP handshake to port 25 is followed by the SMTP dialogue (`220`, `EHLO`, `MAIL FROM`, `RCPT TO`, `DATA`, `QUIT`) and the TCP close. Your To, Subject, and Message fields appear in the dialogue.

**Video streaming.** A TCP handshake is followed by a manifest request and two video segment requests at the chosen quality, then the close. Segment responses are shown as full-size 1460 B segments.

---

## How the transport layer is modeled

### TCP

Each connection keeps a small state object with the client and server ports, each side's next sequence number, and the window size. Every segment is built by one function, `seg()`, which applies these rules:

- **Seq** is the sender's next sequence number.
- **Ack** is the peer's next sequence number. It is only shown when the `ACK` flag is set.
- **Sequence advance:** after sending, the sender's seq increases by the payload length. `SYN` and `FIN` each consume one sequence number even with no payload.
- **Flags:** handshake and teardown use `SYN`, `SYN,ACK`, `FIN,ACK`, and `ACK`. Application data is sent as `PSH,ACK`.
- **Window:** fixed at 64240.

### UDP

UDP datagrams carry source port, destination port, length (payload + 8-byte header), and a checksum. There is no connection state. The DNS reply returns to the client's ephemeral port (53012), which shows how the OS finds the waiting application.

### Simplifications

These are deliberate, to keep the lab readable:

- Initial sequence numbers are fixed (client 1000, server 5000) instead of random.
- There is no retransmission, packet loss, congestion control, or TCP options.
- Pure ACKs for every data segment are omitted. Acknowledgments ride on the next data segment.
- The 4-way close is shown as 3 steps because the server's ACK and FIN are combined.
- The SMTP scenario advertises `STARTTLS` but does not perform it.
- IP addresses are documentation and private ranges (RFC 5737 and RFC 1918). The network layer is not modeled yet.

---

## Code structure

The project is a single HTML file with inline CSS and JavaScript.

| Section | Purpose |
|---|---|
| CSS variables | Light and dark theme tokens, including `--proto-tcp` and `--proto-udp` for the transport badges |
| `step()` | Builds one log step from direction, app data, transport header, endpoints, and a note |
| `mkFlow()`, `seg()` | Create a TCP connection and build correctly numbered segments |
| `udp()` | Builds a UDP datagram |
| `tcpOpen()`, `tcpClose()`, `tcpData()` | Reusable TCP building blocks: handshake, teardown, and app data in a segment |
| `seqBrowse()`, `seqMail()`, `seqStream()` | Scenario generators that assemble the step lists |
| `renderStep()`, `chips()` | Render the stacked application and transport layers, header chips, and encapsulation bar |
| Playback functions | `startSequence`, `playNextStep`, `stepForward`, `stepBackward`, `replaySequence`, `togglePlayPause` |

---

## Extending it

**Add a scenario.** Write a `seqYourThing()` generator, add an entry to `SERVERS`, add a tab and form in the HTML, and handle it in `triggerAction()`. Use `tcpOpen`, `tcpData`, and `tcpClose` for TCP, or `udp()` for datagram traffic.

**Change timing.** Edit the `setTimeout(playNextStep, 1600)` delay in `playNextStep()`.

**Change window size or ISNs.** Edit `mkFlow()`.

**Add the network layer.** Extend `step()` with an `ip` object (source, destination, TTL, protocol number) and add a third stacked layer in `renderStep()`. The layer and encapsulation structure is already set up for it.

---

## Browser support

Any current browser (Chrome, Edge, Firefox, Safari). The page is responsive and stacks the panels on screens narrower than 900 px.
