+++
date = '2026-05-05T08:00:00-04:00'
draft = false
title = 'Flipper Zero Phase 1 — SDK Setup and Hello World in C'
description = 'Setting up the Flipper Zero development environment on Ubuntu from scratch, understanding the SDK, and writing the first C app that runs on real hardware.'
author = 'Jesse Bentley'
tags = ['flipper-zero', 'c', 'embedded', 'sdk', 'ubuntu', 'ufbt']
categories = ['flipper-zero']
series = ['flipper-zero-build']
showToc = true
tocopen = false
+++

## The Goal

Get the Flipper Zero SDK installed on a fresh Ubuntu machine, understand
the basic structure of a Flipper app, and write a C program that
actually runs on the device. Not flash someone else's firmware. Not
install a pre-built app. Write code, compile it, and see it run on
real hardware.

I want to know how it works. I want to see what I can make it do.
And I want to learn embedded systems properly and not just use tools
someone else built.

---

## What I Expected

Phase 1 was supposed to be straightforward. Install the SDK, generate
a template, make a few changes, compile, done. The kind of setup
session that takes an hour and gives you something to build on.

It was mostly that. Mostly.

---

## What I Ran Into

**Setting up ufbt on Ubuntu had a few small hurdles.**

The Flipper build tool is called ufbt a micro Flipper Build Tool.
Installing it via pip worked fine but the executable landed in
`~/.local/bin/` which wasn't in the PATH by default. One line in
`.bashrc` fixed it:

```bash
echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc
```

The Hugo snap install had a similar PATH quirk earlier in the week.
At this point I'm starting to recognize the pattern that new tool
installs, isn't immediately available in terminal, needs a PATH fix.
Getting faster at spotting it.

**USB permissions on Linux.**

When I first tried to deploy to the Flipper I got a permission
denied error on `/dev/ttyACM0`. Linux doesn't give regular users
access to serial devices by default. One command and a logout fixed it:

```bash
sudo usermod -a -G dialout jesse
```

Adding your user to the `dialout` group gives permission to access
serial and USB devices. Log out, log back in, permission granted.

**The Apps folder was missing.**

This is where the real troubleshooting started. The app compiled
and transferred successfully but I couldn't find it on the Flipper.
No Apps folder in the menu.

First instinct — I hadn't formatted the SD card. The Flipper needs
a formatted SD card to store external apps. Fixed that through
Settings on the device. Still no Apps folder.

Second look — firmware was outdated. The Flipper was running 0.64.5.
The current release is significantly newer and the Apps menu structure
changed between versions. Updated the firmware directly from the
Latitude via ufbt:

```bash
ufbt flash_usb
```

That's the cleaner approach anyway. The web updater on Flipper's
site downloads the wrong OS package depending on how it detects
your system. Flashing directly via ufbt uses exactly what the SDK
expects, no ambiguity.

The firmware update UI on the Flipper itself was a nice touch with
a clean progress screen, reboot animation, and back to the main
menu. Small thing but it showed attention to the user experience
even on a developer tool.

After the update the Apps folder was there.

**The code itself.**

I haven't touched C since the early 2000s. Coming back to it after
that long with a specific goal, embedded systems, real hardware,
a tool I actually want to build, felt different than learning it
originally did.

The generated template from ufbt gave me something to read before
I wrote anything. Understanding what was already there before
changing it. Every line explained before moving on. That approach
paid off and when it came time to add the screen output and input
handling I knew what I was adding and where it fit.

The app has two key parts working together.

This is the draw callback which is the function that writes to the screen:

```c
static void draw_callback(Canvas* canvas, void* ctx) {
    UNUSED(ctx);
    canvas_clear(canvas);
    canvas_set_font(canvas, FontPrimary);
    canvas_draw_str(canvas, 10, 20, "Hello from Jesse");
    canvas_set_font(canvas, FontSecondary);
    canvas_draw_str(canvas, 10, 35, "Flipper Zero SDK");
    canvas_draw_str(canvas, 10, 50, "Phase 1 Complete");
}
```

It clears the screen, sets the font, and draws three lines of text.
This function doesn't run on its own and it gets called automatically
by the GUI system every time the screen needs to be redrawn. The
developer doesn't control when it runs, just what it draws when it does.

This is the main loop which is the heartbeat that actually keeps the app alive:

```c
InputEvent event;
bool running = true;

while(running) {
    if(furi_message_queue_get(event_queue, &event, 100) == FuriStatusOk) {
        if(event.type == InputTypePress && event.key == InputKeyBack) {
            running = false;
        }
    }
}
```

Without this the app would draw to the screen once and immediately
terminate. The loop runs continuously checking for button presses.
When it detects the back button it sets `running` to false which
breaks the loop and exits cleanly. The draw callback paints the
picture. The main loop keeps the lights on.

---

## What the Outcome Was

The Apps folder appeared after the firmware update. The compiled
app was sitting in Examples right where ufbt put it. Opened it,
saw the three lines of text on screen, pressed back to exit. Clean.

I haven't written C in over twenty years. Seeing it run on hardware
tonight felt genuinely good. The proof of concept is there. The
motivation to keep going is there. The project is real now in a
way it wasn't before the first compile.

Everything is committed to GitHub. The foundation for Phase 2 is ready.

---

## What's Next

Components arrive this weekend the ESP32-CAM, IR LEDs, NRF24L01,
GPS module, and the CrowPanel 3.5" touchscreen. Phase 2 begins
immediately: the WiFi device scanner. The Flipper connects to the
ESP32 WiFi board via UART and scans for nearby devices. The goal
is detecting the kind of hardware hidden cameras use to phone home.

The build is underway.