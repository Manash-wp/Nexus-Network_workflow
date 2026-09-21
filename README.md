# 🌐 Nexus Network Lab
**Application-Layer Protocol Visualizer**

Nexus Network Lab is an interactive, browser-based educational tool designed to help students, developers, and IT professionals visualize how application-layer protocols work under the hood. It simulates the step-by-step network exchanges between a client and a server, breaking down complex handshakes and data transfers into easy-to-understand logs.

## ✨ Features

* **Interactive Visualization:** Watch packets travel between the client and server as the network sequence progresses.
* **Detailed Protocol Logs:** Inspect simulated DNS queries, TCP handshakes, TLS 1.3 negotiations, HTTP/1.1 requests, and SMTP commands.
* **Playback Controls:** Fully control the simulation. Pause, step forward, step backward, or replay the sequence to study each step at your own pace.
* **Modern UI:** Features a sleek, responsive Deep Purple & Teal interface with a built-in Light/Dark mode toggle.
* **Zero Dependencies:** Built entirely with Vanilla HTML, CSS, and JavaScript. 
* **Safe & Local:** The app is purely a frontend simulation. It does not make any actual external network requests, making it 100% safe to run locally.

## 🔍 Scenarios Covered

1. **Web Browsing (HTTP/TLS/DNS):** Simulates a user navigating to a URL. Shows DNS resolution, TCP connection, TLS encryption setup, and the HTTP GET request/response cycle.
2. **Sending Mail (SMTP):** Simulates sending an email to a mail server. Shows the initial SMTP greeting, EHLO, STARTTLS, and the standard MAIL FROM / RCPT TO / DATA sequence.
3. **Video Streaming (DASH/HTTP):** Simulates a client requesting adaptive bitrate video segments, showing how a client requests manifest files and subsequent video chunks.

## 🚀 How to Use

Because Nexus Network Lab has no external dependencies, running it is incredibly simple:

1. Download or copy the code into a file named `index.html`.
2. Double-click `index.html` to open it in any modern web browser (Chrome, Firefox, Safari, Edge).
3. Select a scenario from the left panel.
4. Customize the input fields (URL, email subject, video quality) if desired.
5. Click the primary action button to start the simulation.

## 🛠️ Customization

If you want to modify the app for your own educational scenarios, you can easily add new sequences in the JavaScript section. Look for the `seqBrowse`, `seqMail`, and `seqStream` functions to see how steps are constructed using the `step()` function:

```javascript
// Example of a custom step
steps.push(step('out', 'PROTOCOL', `Your message here`, CLIENT, server));
