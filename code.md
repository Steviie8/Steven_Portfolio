---
layout: "page"
title: Code
permalink: /code/
---
/* Basic Body/Page Styles */
body {
    font-family: Arial, sans-serif; /* Choose a clean font */
    line-height: 1.6; /* Improve readability */
    margin: 0;
    padding: 0;
    background-color: #f4f4f4; /* Light grey background */
    color: #333; /* Dark grey text */
}

/* Container for content - helps center and limit width */
.container {
    max-width: 960px; /* Max width for content */
    margin: 20px auto; /* Center the container with some top/bottom margin */
    padding: 20px;
    background-color: #fff; /* White background for content area */
    box-shadow: 0 0 10px rgba(0,0,0,0.1); /* Subtle shadow */
    border-radius: 8px; /* Slightly rounded corners */
}

/* Header & Navigation Styles */
header {
    background-color: #333; /* Dark background for header */
    color: #fff;
    padding: 1rem 0;
    text-align: center;
}

nav ul {
    list-style: none; /* Remove bullet points */
    padding: 0;
    margin: 0;
    display: flex; /* Make nav items horizontal */
    justify-content: center; /* Center nav items */
}

nav ul li {
    margin: 0 15px; /* Spacing between nav items */
}

nav a {
    color: #fff;
    text-decoration: none; /* Remove underline from links */
    font-weight: bold;
    transition: color 0.3s ease; /* Smooth color transition on hover */
}

nav a:hover {
    color: #ddd; /* Lighter color on hover */
}

/* Headings */
h1, h2, h3 {
    color: #2c3e50; /* Darker color for headings */
    margin-top: 1.5em;
    margin-bottom: 0.5em;
}

/* Paragraphs */
p {
    margin-bottom: 1em;
}

/* Links */
a {
    color: #3498db; /* A blue color for links */
    text-decoration: none;
}

a:hover {
    text-decoration: underline;
}

/* Code Block Styling (Crucial for code.md) */
pre {
    background-color: #282c34; /* Dark background for code blocks */
    color: #abb2bf; /* Light text color for code */
    padding: 1em;
    border-radius: 5px;
    overflow-x: auto; /* Allow horizontal scrolling for long lines */
    font-family: 'Courier New', monospace; /* Monospaced font for code */
    font-size: 0.9em;
    line-height: 1.4;
}

/* For inline code (if you use `some code`) */
code {
    background-color: #e0e0e0;
    padding: 0.2em 0.4em;
    border-radius: 3px;
    font-family: 'Courier New', monospace;
    font-size: 0.9em;
}

/* Table Styling (for your engineer table) */
table {
    width: 100%;
    border-collapse: collapse;
    margin: 1em 0;
}

table th, table td {
    border: 1px solid #ddd;
    padding: 8px;
    text-align: left;
}

table th {
    background-color: #f2f2f2;
    font-weight: bold;
}

table tr:nth-child(even) {
    background-color: #f9f9f9;
}

/* Footer */
footer {
    text-align: center;
    padding: 1em 0;
    margin-top: 2em;
    border-top: 1px solid #eee;
    color: #777;
    font-size: 0.8em;
}
## My code
# Second Milestone Code
My code for the second milestone utilizes multiple Python files, along with the main file, which calls on the other files to run. This allows me to display multiple objects on the screen.
Main Code
```python 
import board
import digitalio
import time
import displayio
import terminalio
from os import getenv
from adafruit_display_text.label import Label
from adafruit_bitmap_font import bitmap_font
from adafruit_matrixportal.network import Network
from adafruit_matrixportal.matrix import Matrix
from time_1 import run 
from Flow import loop, setup, cleanup
import val_test
import gc

ssid = getenv("CIRCUITPY_WIFI_SSID")
password = getenv("CIRCUITPY_WIFI_PASSWORD")
if None in [ssid, password]:
    raise RuntimeError("Missing WiFi credentials in settings.toml")

matrix = Matrix()
display = matrix.display
font_4x6 = bitmap_font.load_font("Roboto-Regular-6pt.bdf")
valorant_group = val_test.setup_valorant_display(display, font=font_4x6)
try:
    val_test._connection_status = "Connecting..."
except AttributeError:
    print("Warning: _connection_status not found in val_test.py. Ensure it's accessible or use a setter.")

display.root_group = valorant_group
display.refresh()

network = Network(status_neopixel=board.NEOPIXEL, debug=False)
try:
    network.connect()
    print("Connected to Wi-Fi")
    try:
        val_test._connection_status = "Connected"
    except AttributeError:
        pass
except Exception as e:
    print(f"Wi-Fi Error: {e}")
    try:
        val_test._connection_status = "Wi-Fi Error"
    except AttributeError:
        pass

clock_group = displayio.Group()
clock_bitmap = displayio.Bitmap(64, 32, 2)
clock_palette = displayio.Palette(4)
clock_palette[0] = 0x000000
clock_palette[1] = 0xFF0000
clock_palette[2] = 0xCC4000
clock_palette[3] = 0x85FF00
clock_tile_grid = displayio.TileGrid(clock_bitmap, pixel_shader=clock_palette)
clock_group.append(clock_tile_grid)
clock_font = bitmap_font.load_font("/IBMPlexMono-Medium-24_jep.bdf")
clock_label = Label(clock_font)
clock_group.append(clock_label)

def update_time(*, hours=None, minutes=None, show_colon=False):
    now = time.localtime()
    if hours is None:
        hours = now[3]
    if hours >= 18 or hours < 6:
        clock_label.color = clock_palette[1]
    else:
        clock_label.color = clock_palette[3]
    if hours > 12:
        hours -= 12
    elif not hours:
        hours = 12
    if minutes is None:
        minutes = now[4]
    colon = ":" if show_colon or now[5] % 2 else " "
    clock_label.text = f"{hours}{colon}{minutes:02d}"
    bbx, bby, bbwidth, bbh = clock_label.bounding_box
    clock_label.x = round(display.width / 2 - bbwidth / 2)
    clock_label.y = display.height // 2

button_up = digitalio.DigitalInOut(board.BUTTON_UP)
button_up.direction = digitalio.Direction.INPUT
button_up.pull = digitalio.Pull.UP
button_down = digitalio.DigitalInOut(board.BUTTON_DOWN)
button_down.direction = digitalio.Direction.INPUT
button_down.pull = digitalio.Pull.UP

mode = 1
last_check = None
last_up = True
last_down = True
last_mode = -1

while True:
    if mode != last_mode:
        print(f"Switching from Mode {last_mode} to Mode {mode}")
        if last_mode == 1:
            pass
        elif last_mode == 2:
            cleanup()
            print("Cleaned up Flow Mode.")
        elif last_mode == 3:
            print("Valorant Mode has no dedicated cleanup function (objects are persistent).")
        if mode == 1:
            display.root_group = clock_group
            print("Switched to Clock Mode")
        elif mode == 2:
            display.root_group = setup(display)
            print("Switched to Flow Mode")
        elif mode == 3:
            display.root_group = valorant_group
            print("Switched to Valorant Mode")
        gc.collect()
    if mode == 1:
        last_check = run(update_time, network, last_check)
        gc.collect
    elif mode == 2:
        loop()
        gc.collect
    elif mode == 3:
        val_test.get_info(network)
    last_mode = mode
    current_up = button_up.value
    current_down = button_down.value
    if not current_up and last_up and mode < 3:
        mode +=1
        print("Up button pressed")
    if not current_down and last_down and mode > 0:
        mode -= 1
        print("Down button pressed")
    last_up = current_up
    last_down = current_down
    time.sleep(0.05)
```

# First Milestone Code
My code
```python 
# SPDX-FileCopyrightText: 2020 John Park for Adafruit Industries
#
# SPDX-License-Identifier: MIT

# Metro Matrix Clock
# Runs on Airlift Metro M4 with 64x32 RGB Matrix display & shield

from os import getenv
import time
import board
import displayio
import terminalio
from adafruit_display_text.label import Label
from adafruit_bitmap_font import bitmap_font
from adafruit_matrixportal.network import Network
from adafruit_matrixportal.matrix import Matrix

BLINK = True
DEBUG = False

# Get WiFi details, ensure these are setup in settings.toml
ssid = getenv("CIRCUITPY_WIFI_SSID")
password = getenv("CIRCUITPY_WIFI_PASSWORD")

if None in [ssid, password]:
    raise RuntimeError(
        "WiFi settings are kept in settings.toml, "
        "please add them there. The settings file must contain "
        "'CIRCUITPY_WIFI_SSID', 'CIRCUITPY_WIFI_PASSWORD', "
        "at a minimum."
    )

print("    Metro Minimal Clock")
print("Time will be set for {}".format(getenv("timezone")))

# --- Display setup ---
matrix = Matrix()
display = matrix.display
network = Network(status_neopixel=board.NEOPIXEL, debug=False)

# --- Drawing setup ---
group = displayio.Group()  # Create a Group
bitmap = displayio.Bitmap(64, 32, 2)  # Create a bitmap object,width, height, bit depth
color = displayio.Palette(4)  # Create a color palette
color[0] = 0x000000  # black background
color[1] = 0xFF0000  # red
color[2] = 0xCC4000  # amber
color[3] = 0x85FF00  # greenish

# Create a TileGrid using the Bitmap and Palette
tile_grid = displayio.TileGrid(bitmap, pixel_shader=color)
group.append(tile_grid)  # Add the TileGrid to the Group
display.root_group = group

if not DEBUG:
    font = bitmap_font.load_font("/IBMPlexMono-Medium-24_jep.bdf")
else:
    font = terminalio.FONT

clock_label = Label(font)


def update_time(*, hours=None, minutes=None, show_colon=False):
    now = time.localtime()  # Get the time values we need
    if hours is None:
        hours = now[3]
    if hours >= 18 or hours < 6:  # evening hours to morning
        clock_label.color = color[1]
    else:
        clock_label.color = color[3]  # daylight hours
    if hours > 12:  # Handle times later than 12:59
        hours -= 12
    elif not hours:  # Handle times between 0:00 and 0:59
        hours = 12

    if minutes is None:
        minutes = now[4]

    if BLINK:
        colon = ":" if show_colon or now[5] % 2 else " "
    else:
        colon = ":"

    clock_label.text = "{hours}{colon}{minutes:02d}".format(
        hours=hours, minutes=minutes, colon=colon
    )
    bbx, bby, bbwidth, bbh = clock_label.bounding_box
    # Center the label
    clock_label.x = round(display.width / 2 - bbwidth / 2)
    clock_label.y = display.height // 2
    if DEBUG:
        print("Label bounding box: {},{},{},{}".format(bbx, bby, bbwidth, bbh))
        print("Label x: {} y: {}".format(clock_label.x, clock_label.y))


last_check = None
update_time(show_colon=True)  # Display whatever time is on the board
group.append(clock_label)  # add the clock label to the group

while True:
    if last_check is None or time.monotonic() > last_check + 3600:
        try:
            update_time(
                show_colon=True
            )  # Make sure a colon is displayed while updating
            network.get_local_time()  # Synchronize Board's clock to Internet
            last_check = time.monotonic()
        except RuntimeError as e:
            print("Some error occured, retrying! -", e)

    update_time()
    time.sleep(1)

```


[Back to Home](index.html)
