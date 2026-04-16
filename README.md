# Pi e-Paper Image Gallery

A Raspberry Pi web application that lets users upload images through a browser, processes them for display, and renders them on a Waveshare 2.7" e-Paper screen — with an optional approval queue, live queue stats, and automatic image orientation handling.

---

## Overview

This project runs a Flask web server on a Raspberry Pi. Users on the local network visit the page, upload an image, and it gets added to a display queue. Images are automatically resized and rotated to fit the e-Paper display's resolution. An optional Tkinter approval window lets an admin preview and approve or decline each submission before it reaches the screen.

---

## Features

- **Browser-based upload** — any device on the local network can submit images via the web interface
- **Approval mode** — optional Tkinter window for previewing and approving/declining images before display
- **Smart image processing** — auto-detects portrait vs landscape orientation and resizes to the correct e-Paper resolution (176×264 or 264×176)
- **Queue management** — thread-safe queue ensures images display one at a time with a 10-second interval between each
- **Live queue stats** — estimated wait time, queue length, and total images displayed pushed to the browser via Server-Sent Events (SSE)
- **Threaded architecture** — Flask server, display queue, and approval queue each run on independent threads

---

## Tech Stack

- **Language:** Python 3
- **Hardware:** Raspberry Pi + Waveshare 2.7" e-Paper display
- **Web:** Flask + Server-Sent Events (SSE)
- **Image processing:** Pillow (PIL)
- **GUI (approval mode):** Tkinter
- **Concurrency:** `threading` + `queue` modules

---

## Project Structure

```
pi-epaper-image-gallery/
├── main.py          # Flask app, queue logic, e-Paper display handler, approval Tkinter window
├── requirements.txt # Python dependencies
└── templates/
    └── index.html   # Upload form and live queue stats page
```

---

## How It Works

1. Flask serves `index.html` — a simple upload form with a live queue status panel
2. On image submit, the file is saved to `images/` and placed in either the display queue or approval queue depending on the `approval` flag
3. If approval is enabled, a Tkinter window shows each submitted image — admin clicks **Approve** (adds to display queue) or **Decline** (discards)
4. The display queue handler picks images one by one, determines orientation, resizes with Pillow, and pushes the buffer to the e-Paper via the Waveshare `epd2in7` library
5. Queue length and estimated time remaining stream to the browser every second via SSE

---

## Setup

**1. Clone the repo on your Raspberry Pi**
```bash
git clone https://github.com/candtk/pi-epaper-image-gallery.git
cd pi-epaper-image-gallery
```

**2. Install Waveshare e-Paper library**
```bash
sudo apt-get update
sudo apt-get install -y python3-pip libopenjp2-7 libatlas-base-dev python3-pil
git clone https://github.com/waveshare/e-Paper ~/e-Paper
cp -r ~/e-Paper/RaspberryPi_JetsonNano/python/lib/waveshare_epd .
rm -rf ~/e-Paper
```

**3. Install Python dependencies**
```bash
pip3 install -r requirements.txt
```

**4. Run**
```bash
python3 main.py
```

The server starts at `http://0.0.0.0:5000`. Access it from any device on the same network using your Pi's IP address:
```
http://<raspberry-pi-ip>:5000
```

Find your Pi's IP with:
```bash
hostname -I
```

---

## Configuration

At the bottom of `main.py`, tweak the startup options:

```python
InitiateApp = ImageDisplayWeb(host="0.0.0.0", port=5000, approval=True)
```

| Option | Default | Description |
|---|---|---|
| `host` | `0.0.0.0` | Network interface to bind to |
| `port` | `5000` | Port the web server listens on |
| `approval` | `True` | Enable/disable the Tkinter approval window |

---

## License

MIT
