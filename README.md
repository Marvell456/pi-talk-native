# PiTalk Local

A voice assistant I made with a Raspberry Pi 5. It runs offline. You hold a button, talk, and it shows a reply on a small screen.

I made this because I wanted to see if I could run a chatbot without using OpenAI or Google. It's not fast, but it works.

[Demo Video](./DEMO.md)

---

## What it uses

- Raspberry Pi 5 (8GB)
- USB microphone
- 1.3" OLED screen (I2C)
- Push button
- faster-whisper for speech to text
- Ollama with qwen2.5:1.5b

---

## How it works

1. Hold the button
2. It records audio
3. faster-whisper turns it into text
4. Text goes to Ollama
5. Reply shows on the OLED

---

## Things that went wrong

- **Audio was choppy and kept crashing.** I was trying to record and run the AI on the same thread. Fixed it by using a queue and a separate thread for audio.
- **Button was backwards.** I used a pull-up resistor, so the pin reads LOW when pressed. My code was checking for HIGH. Changed it to check for LOW.
- **OLED showed random pixels.** I used the wrong driver. My screen is SH1106, not SSD1306. Swapped the driver and it worked.
- **The AI is slow.** qwen2.5:1.5b is small but still takes a few seconds. It also gives dumb answers sometimes. That's just how it is on a Pi.

---

## Setup

1. Install packages:
```bash
sudo apt update
sudo apt install -y python3-pip python3-dev libportaudio2 libasound2-dev fonts-dejavu git
```

2. Install Ollama and pull the model:

```bash
curl -fsSL https://ollama.com/install.sh | sh
ollama run qwen2.5:1.5b
```

3. Make a venv and install Python stuff:

```bash
python3 -m venv --system-site-packages env
source env/bin/activate
pip install sounddevice numpy scipy requests faster-whisper luma.oled gpiod
```

4. Run it:

```bash
python chatbot.py
```

Make sure ollama serve is running first.

---

Wiring

· Button: GPIO 24 to GND
· OLED: SDA/SCL to I2C pins, address 0x3C
· Mic: USB

---

Notes

This is just a learning project. I'm not an expert. I followed a lot of tutorials and broke things a few times. It's slow and not perfect, but it was fun to build.