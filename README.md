PiTalk Local

A voice assistant I built on a Raspberry Pi 5. No cloud, no APIs, everything runs on the Pi itself.

I wanted to see if I could make something like Alexa but completely offline. Turns out you can, it's just kinda slow and takes some fiddling to get working.

Go to DEMO.md for the demo video

---

What It Does

You hold a button, talk into a mic, and the Pi figures out what you said, thinks of a response, and shows it on a tiny screen. All locally. No internet needed after setup.

---

Stuff I Used

· Raspberry Pi 5 (8GB)
· Raspberry Pi OS Lite - lighter = more RAM for the AI
· USB microphone - just a cheap plug and play one
· 1.3 inch OLED screen - SH1106 chip, connected via I2C
· Arcade button - wired to GPIO pin 24
· faster-whisper - for speech to text (tiny model, int8)
· Ollama with qwen2.5:1.5b - the actual "brain"

---

Problems I Ran Into

1. Mic kept crashing

First time I tried recording, everything froze and I got a million paInputOverflowed errors.

Turns out Python was trying to do everything on one thread. The AI part would hog all the CPU while the mic kept dumping audio into a queue that never got emptied. Also the mic didn't like 16000 Hz for some reason.

Fix: Told the Pi to use all 4 cores with environment variables, and made the audio recording run on its own separate loop so it doesn't get blocked. Also switched to 44100 Hz which the mic actually supports.

2. Button was backwards

Held the button down = nothing. Let go = it records forever.

I set up the GPIO pin with a pull-up resistor, which means the pin reads HIGH when idle and LOW when you press the button. My code was checking for HIGH to start recording. So it was literally doing the opposite of what I wanted.

Fix: Changed the check to look for Value.INACTIVE instead.

3. Screen showed garbage

First time I tried displaying text, I got random pixels and flickering lines. Looked like TV static.

Most tutorials assume you have an SSD1306 screen. Mine is SH1106. They have the same I2C address but completely different memory layouts, so all the commands were going to the wrong places.

Fix: Used the sh1106 driver from luma.oled instead of the default one. Also added a device.clear() on startup to wipe any leftover garbage.

---

How To Set It Up

1. Install stuff

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y python3-pip python3-dev libportaudio2 libasound2-dev fonts-dejavu git
```

2. Get Ollama running

```bash
curl -fsSL https://ollama.com/install.sh | sh
ollama run qwen2.5:1.5b
```

Type /bye to exit after it downloads.

3. Set up Python environment

```bash
python3 -m venv --system-site-packages env
source env/bin/activate

pip install --upgrade pip
pip install sounddevice numpy scipy requests faster-whisper luma.oled gpiod
```

4. Run it

```bash
python chatbot.py
```

The code is already in this repo. Make sure ollama serve is running in another terminal first.

---

Wiring

· Button: GPIO 24 to ground (internal pull-up handles the rest)
· OLED: SDA/SCL to the I2C pins, address 0x3C
· Mic: just plug it in via USB

---

What I Learned

· Running AI on small devices is hard. You have to use tiny models and quantize them down or the Pi just chokes.
· The new libgpiod library is way better than the old RPi.GPIO stuff but the docs are confusing.
· Audio is annoying. Sample rates, buffers, threading... lots of things can go wrong.

---

Notes

This isn't meant to be a product or anything. Just a project I made because I was curious if it would work. It's slow (takes a few seconds to respond) but it works and that's cool enough for me.
