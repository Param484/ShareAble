<div align="center">

# ShareAble

### Share files instantly — no servers, no sign-up, no limits.

**ShareAble** is a browser-based, peer-to-peer file sharing app built on WebRTC. Files travel directly between devices — nothing is uploaded to any server. Open the page on two devices, share a 6-character Room ID (or scan a QR code), and start transferring.

[Live Demo](#) · [Report a Bug](https://github.com/yourusername/shareable/issues) · [Request a Feature](https://github.com/yourusername/shareable/issues)

</div>

---

## What it does

- **Zero-server file transfer** — files go directly device-to-device over WebRTC. No cloud storage, no upload limits.
- **Any file type, any size** — PDFs, videos, ZIP archives, executables — anything goes.
- **Live progress** — real-time transfer speed, percentage, and ETA synced between sender and receiver.
- **Radar discovery** — receivers auto-detect nearby senders on the same network without typing any ID.
- **Built-in chat** — send text messages alongside file transfers with delivery receipts and emoji reactions.
- **QR code connect** — generate a QR from your Room ID; the other device scans and connects automatically.
- **File queue** — drag in multiple files and send them all in one session, one after another.
- **Cancel mid-transfer** — press X at any time; the transfer stops immediately on both sides with no partial file saved.
- **Dark & light theme** — auto-detected from system preference, toggleable at any time.
- **Mobile optimised** — dedicated layout with large tap targets for phones and tablets.
- **No install** — a single HTML file. Open it in any modern browser and it works.

---

## Screenshots



---

## Getting Started

### Option A — Just open the file

Download `index.html` and open it in your browser. That's it. Works on Chrome, Edge, Firefox, and Safari.

```bash
# Clone the repo
git clone https://github.com/yourusername/shareable.git
cd shareable

# Open directly
open index.html          # macOS
start index.html         # Windows
xdg-open index.html      # Linux
```

### Option B — Serve locally (recommended for QR auto-connect)

Running over `http://` instead of `file://` unlocks QR-based auto-connect — the scanned device opens the app and fills in the Room ID automatically.

```bash
# Using Node.js
npx serve .

# Using Python
python -m http.server 8080
```

Then open `http://localhost:8080` (or `http://YOUR_LOCAL_IP:8080` on another device on the same network).

### Option C — Deploy for free

Host the single `index.html` file on any static host:

| Platform | Steps |
|---|---|
| **GitHub Pages** | Push to a repo → Settings → Pages → Deploy from `main` branch |
| **Netlify** | Drag the folder to [app.netlify.com/drop](https://app.netlify.com/drop) |
| **Vercel** | `npx vercel` in the project folder |
| **Cloudflare Pages** | Connect repo or drag-and-drop in the dashboard |

Once deployed, paste the URL in **⚙️ Settings → QR Auto-Connect URL** so QR codes auto-open and connect.

---

## How to Use

### Sending a file

1. Open ShareAble on your device and pick **Sender**.
2. Your **Room ID** (6 characters) appears at the top — share it with the receiver via copy, QR code, or verbally.
3. The receiver enters your Room ID and clicks **Connect**.
4. Drag files into the drop zone (or tap to browse on mobile).
5. Click **Send** — progress appears live on both sides.

### Receiving a file

1. Open ShareAble and pick **Receiver**.
2. Enter the sender's Room ID and click **Connect** — or use Radar if they're on the same network.
3. Incoming files download automatically when the transfer completes.

### QR Code connect

1. Click the **QR** button next to your Room ID.
2. The other device scans the QR with its camera app.
3. ShareAble opens on that device and connects automatically (requires the app to be served over `http/https`).

---

## How it works

```
Device A (Sender)                    Device B (Receiver)
      │                                      │
      │  1. Peer registers with PeerJS       │
      │     signaling server (ID exchange)   │
      │◄─────────────────────────────────────►│
      │                                      │
      │  2. WebRTC data channel established  │
      │     (direct P2P, no server relay)    │
      │◄═════════════════════════════════════►│
      │                                      │
      │  3. Files stream as 64 KB chunks     │
      │  ════════════════════════════════════►│
      │                                      │
      │  4. Receiver ACKs bytes back         │
      │  ◄════════════════════════════════════│
      │     (both progress bars in sync)     │
```

**Technology stack:**

| Layer | Technology |
|---|---|
| P2P transport | [WebRTC](https://webrtc.org/) Data Channels |
| Signaling | [PeerJS](https://peerjs.com/) (free public broker) |
| Discovery | Lobby peer + BroadcastChannel |
| QR codes | [qrcode.js](https://github.com/davidshimjs/qrcodejs) |
| Audio | Web Audio API (no audio files) |
| Storage | `localStorage` (profile, theme, stats) |
| Dependencies | None beyond the two CDN scripts above |

**Transfer internals:**

- Chunk size: **64 KB** — tuned for WebRTC's SCTP MTU
- High-water mark: **4 MB** buffered — prevents memory spikes
- Low-water mark: **512 KB** — resumes the pump quickly
- Progress sync: receiver sends `file-ack` messages back to the sender so both bars show the same real number
- Cancel: sets `transferCancelled` flag + calls `AbortController.abort()` on the in-progress `arrayBuffer()` read — no partial file reaches the receiver

---

## Project structure

Everything lives in a single file:

```
shareable/
├── index.html        ← entire app (HTML + CSS + JS, ~4 500 lines)
└── README.md
```

Internal organisation within `index.html`:

```
<head>
  CSS variables (theme tokens)
  Component styles

<body>
  Intro overlay
  App shell (header, role chooser, sender panel, receiver panel)
  Settings panel
  Chat drawer
  Modals (QR, disconnect, "send another")

<script>
  State variables
  initApp / profile / theme
  Peer setup (initPeer, chooseRole)
  Radar / lobby discovery (registerAsSender, startRadar, startLobby)
  File queue (addToQueue, renderQueue, removeFromQueue)
  Sending (startSending, sendNextFile, sendFile, cancelFileByIndex)
  Receiving (handleData, finalizeReceive)
  Chat (sendChatMessage, receiveChatMessage, reactions, ACKs)
  History / session stats
  Disconnect / modals
  Utilities (formatBytes, formatETA, getFileIcon, sounds)
  QR (showQR, getAppBaseUrl, saveAppUrl)
  Intro splash
```

---

## Browser support

| Browser | Support |
|---|---|
| Chrome / Edge 80+ | ✅ Full |
| Firefox 75+ | ✅ Full |
| Safari 15.4+ | ✅ Full |
| iOS Safari 15.4+ | ✅ Full |
| Android Chrome | ✅ Full |
| IE / old Edge | ❌ Not supported |

WebRTC Data Channels and `file.arrayBuffer()` are required. Both are available in all modern browsers.

---

## Privacy

- **No file data ever touches a server.** Files stream directly between browsers via WebRTC.
- The only server contact is the PeerJS signaling broker — it exchanges WebRTC handshake metadata (ICE candidates, SDP) only. It never sees file content.
- Your Room ID, name, and avatar are stored in `localStorage` on your own device only.
- Transfer history is in-memory and disappears when you close the tab.

---

## Known limitations

- Both devices must have the app open simultaneously (no background/offline delivery).
- Very large files (>2 GB) may hit `arrayBuffer()` memory limits on low-RAM devices.
- The PeerJS public signaling server is a free shared service — for production use, self-host a [PeerServer](https://github.com/peers/peerserver).
- Folder transfer is not supported — zip the folder first.

---

## Local development

No build step. Edit `index.html` directly and refresh the browser.

```bash
# Serve with live reload (optional)
npx browser-sync start --server --files "index.html"
```

To use a self-hosted PeerJS server, update the `initPeer()` call in the script:

```js
peer = new Peer(id, {
  host: 'your-peerserver.com',
  port: 443,
  path: '/peerjs',
  secure: true
});
```

---

## Contributing

Pull requests are welcome. For major changes please open an issue first to discuss what you'd like to change.

1. Fork the repo
2. Create your feature branch: `git checkout -b feature/my-feature`
3. Commit your changes: `git commit -m 'Add my feature'`
4. Push: `git push origin feature/my-feature`
5. Open a Pull Request

---

## Author

**Param Joshi**

- X (Twitter): [@Param_Joshi02](https://x.com/Param_Joshi02)
- LinkedIn: [Param Joshi](https://linkedin.com/in/param-joshi)

---

## License

[MIT](LICENSE) — do whatever you want, just keep the credit.

---

<div align="center">

Built with ❤️ and vanilla JavaScript · No frameworks · No build tools · No servers

</div>
