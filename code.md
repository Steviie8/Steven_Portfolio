---
layout: default
title: Code
---

[Back to Home](index.md)

## My code
# Second Milestone Code
My code for the second milestone utilizes multiple Python files, along with the main file, which calls on the other files to run. This allows me to display multiple objects on the screen.

Main.py
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
Val_Test.py
```python 
import time
import displayio
import gc
from adafruit_display_text.label import Label
import terminalio
from adafruit_bitmap_font import bitmap_font
import adafruit_imageload

name = "TenZ"
tag = "00005"
region = "na"
api_key = "HDEV-72f89bbe-b57c-4e42-a5d4-ca19607a5810"
url = f"https://api.henrikdev.xyz/valorant/v1/mmr/{region}/{name}/{tag}"
headers = {"Authorization": api_key}

_valorant_group = None
_valorant_labels = []
_display_obj = None
_fetch_interval = 180
_last_fetch_time = time.monotonic() - _fetch_interval - 1
_rank_image_container_group = None
_connection_status = "Not Connected"
_valorant_font = None

BORDER_COLOR = 0x8f29e3

def get_rank_image_path(rank_name):
    if not rank_name:
        return None
    return f"/ranks/{rank_name.lower().replace(' ', '_')}.bmp"

def create_image_container_with_border(bitmap=None, palette=None):
    image_container_group = displayio.Group()
    border_width = 1
    image_expected_width = 8
    image_expected_height = 8

    border_bmp_width = image_expected_width + (2 * border_width)
    border_bmp_height = image_expected_height + (2 * border_width)
    border_bitmap = displayio.Bitmap(border_bmp_width, border_bmp_height, 1)
    border_palette = displayio.Palette(1)
    border_palette[0] = BORDER_COLOR

    for x in range(border_bmp_width):
        for y in range(border_bmp_height):
            if x < border_width or x >= border_bmp_width - border_width or y < border_width or y >= border_bmp_height - border_width:
                border_bitmap[x, y] = 0

    border_tilegrid = displayio.TileGrid(border_bitmap, pixel_shader=border_palette)
    image_container_group.append(border_tilegrid)

    if bitmap and palette:
        image_tilegrid = displayio.TileGrid(bitmap, pixel_shader=palette)
        image_tilegrid.x = border_width
        image_tilegrid.y = border_width
        image_container_group.append(image_tilegrid)

    image_container_group.x = 34
    image_container_group.y = _valorant_labels[1].y - 4 if len(_valorant_labels) > 1 else 0

    return image_container_group

def update_rank_image(image_path):
    global _rank_image_container_group, _valorant_group

    fallback_container = create_image_container_with_border()
    if _valorant_group is None:
        return

    if not image_path:
        if _rank_image_container_group in _valorant_group:
            _valorant_group.remove(_rank_image_container_group)
        _rank_image_container_group = fallback_container
        _valorant_group.append(_rank_image_container_group)
        gc.collect()
        return

    try:
        bitmap, palette = adafruit_imageload.load(image_path)
        new_image_container = create_image_container_with_border(bitmap, palette)

        if _rank_image_container_group in _valorant_group:
            _valorant_group.remove(_rank_image_container_group)

        _rank_image_container_group = new_image_container
        _valorant_group.append(_rank_image_container_group)
        gc.collect()
    except Exception:
        if _rank_image_container_group not in _valorant_group:
            _rank_image_container_group = fallback_container
            _valorant_group.append(_rank_image_container_group)
            gc.collect()

def setup_valorant_display(display_obj, line_height=9, font=None):
    global _valorant_group, _valorant_labels, _display_obj, _rank_image_container_group, _valorant_font
    _display_obj = display_obj

    if _valorant_group is None:
        _valorant_group = displayio.Group()
        _valorant_labels = []
        _valorant_font = font
        y_position = 6
        if font is None:
            font = terminalio.FONT

        for i in range(6):
            label = Label(font, color=0x8f29e3, text="", x=1, y=y_position + (i * line_height))
            _valorant_group.append(label)
            _valorant_labels.append(label)

        _rank_image_container_group = create_image_container_with_border()
        _valorant_group.append(_rank_image_container_group)
        gc.collect()

    return _valorant_group

def get_valorant_group():
    return _valorant_group

def get_info(network):
    global _valorant_labels, _valorant_group, _display_obj
    global _connection_status, _last_fetch_time, _fetch_interval
    global _rank_image_container_group

    if _valorant_group is None or _display_obj is None:
        return

    now = time.monotonic()
    if now - _last_fetch_time < _fetch_interval:
        if _connection_status != "Connected" and len(_valorant_labels) > 0:
            _valorant_labels[0].text = f"Status: {_connection_status}"
        _display_obj.refresh()
        return

    _last_fetch_time = now

    for i in range(len(_valorant_labels)):
        _valorant_labels[i].text = ""
    if len(_valorant_labels) > 0:
        _valorant_labels[0].text = "Status: Fetching..."
    _display_obj.refresh()

    lines_to_display = []

    try:
        gc.collect()
        response = network.requests.get(url, headers=headers)
        data = response.json()

        if response.status_code == 200 and "data" in data:
            _connection_status = "Connected"
            mmr = data["data"]
            name_tag = f"{mmr.get('name')} #{mmr.get('tag')}"
            rank = mmr.get('currenttierpatched')
            change = mmr.get('mmr_change_to_last_game')

            lines_to_display.append(name_tag)
            lines_to_display.append(rank)
            lines_to_display.append(f"RR: {change if change is not None else 'N/A'}")

            image_path = get_rank_image_path(rank)
            update_rank_image(image_path)
        else:
            _connection_status = "API Error"
            lines_to_display.append(f"Status: {_connection_status}")
            lines_to_display.append(f"Code: {response.status_code}")
            if "errors" in data and data["errors"]:

```
Time_1.py
```python 
import time


def run(update_time, network, last_check, allow_network=True):
    if allow_network and (last_check is None or time.monotonic() > last_check + 3600):
        try:
            update_time(show_colon=True)
            network.get_local_time()
            last_check = time.monotonic()
        except RuntimeError as e:
            print("Some error occurred, retrying! -", e)

    update_time()
    time.sleep(1)
    return last_check

```
Flow.py
```python 
import time
import math
import displayio
import gc # Import gc

# -- User Config --
SINGULARITIES = (
    ('freestream', None, (1, 0)),
    ('vortex', (26, 16), 6),
    ('vortex', (38, 16), -6),
)
SEEDS = [(0, y) for y in range(0, 32, 3)]
MATRIX_WIDTH = 64
MATRIX_HEIGHT = 32
BACK_COLOR = 0x000000
SING_COLOR = 0xADAF00
HEAD_COLOR = 0x00FFFF
TAIL_COLOR = 0x000A0A
TAIL_LENGTH = 10
DELAY = 0.01

# Globals to be assigned in setup()
display = None
group = None # This will be the main displayio.Group for Flow mode
bitmap = None
palette = None
tile_grid = None

STREAMLINES = []
HEADS = []

def compute_velocity(x, y):
    vx = vy = 0
    for s in SINGULARITIES:
        if s[0] == 'freestream':
            vx += s[2][0]
            vy += s[2][1]
        else:
            dx = x - s[1][0]
            dy = y - s[1][1]
            r2 = dx * dx + dy * dy
            if r2 == 0:
                continue
            if s[0] == 'source':
                vx += s[2] * dx / r2
                vy += s[2] * dy / r2
            elif s[0] == 'vortex':
                vx -= s[2] * dy / r2
                vy += s[2] * dx / r2
            elif s[0] == 'doublet':
                vx += s[2] * (dy * dy - dx * dx) / (r2 * r2)
                vy -= s[2] * (2 * dx * dy) / (r2 * r2)
    return vx, vy

def compute_streamlines():
    global STREAMLINES
    STREAMLINES = [] # Clear existing streamlines before recomputing
    for seed in SEEDS:
        streamline = []
        x, y = seed
        px = round(x)
        py = round(y)
        vx, vy = compute_velocity(x, y)
        streamline.append(((px, py), (vx, vy)))
        steps = 0
        while x < MATRIX_WIDTH and steps < 2 * MATRIX_WIDTH:
            nx = round(x)
            ny = round(y)
            if nx != px or ny != py:
                streamline.append(((nx, ny), (vx, vy)))
                px = nx
                py = ny
            vx, vy = compute_velocity(x, y)
            x += vx
            y += vy
            steps += 1
        STREAMLINES.append(streamline)

def show_singularities():
    for s in SINGULARITIES:
        try:
            x, y = s[1]
            bitmap[round(x), round(y)] = 1
        except:
            pass

def show_streamlines():
    for sl, head in enumerate(HEADS):
        try:
            streamline = STREAMLINES[sl]
            index = round(head)
            length = min(index, TAIL_LENGTH)
            for data in streamline[index - length:index]:
                x, y = data[0]
                bitmap[round(x), round(y)] = 3
            x, y = streamline[index][0]
            bitmap[round(x), round(y)] = 2
        except:
            pass

def animate_streamlines():
    reset_heads = True
    for sl, head in enumerate(HEADS):
        streamline = STREAMLINES[sl]
        index = round(head)
        if index < len(streamline):
            vx, vy = streamline[index][1]
            reset_heads = False
        else:
            vx, vy = streamline[-1][1]
        HEADS[sl] += math.sqrt(vx * vx + vy * vy)
    if reset_heads:
        for index in range(len(HEADS)):
            HEADS[index] = 0

def update_display():
    global display
    if display is None:
        print("ERROR: Flow.py's global 'display' is None in update_display. Skipping refresh.")
        return
    if not hasattr(display, 'auto_refresh'):
        print(f"ERROR: Flow.py's global 'display' (type: {type(display)}) lacks 'auto_refresh' attribute. Skipping refresh.")
        print(f"Dir of display: {dir(display)}")
        return

    display.auto_refresh = False
    bitmap.fill(0)
    show_singularities()
    show_streamlines()
    display.auto_refresh = True

def setup(display_obj):
    global display, group, bitmap, palette, tile_grid, HEADS

    display = display_obj # Always update the global 'display' reference

    if group is not None:
        print("Flow Mode resources already exist. Recomputing streamlines and resetting heads for consistency.")
        compute_streamlines() # Recompute streamlines
        HEADS = [0] * len(STREAMLINES) # Reset heads
        gc.collect()
        return group # Return existing group

    print("Setting up Flow Mode resources...")
    group = displayio.Group()

    bitmap = displayio.Bitmap(display.width, display.height, 4)
    palette = displayio.Palette(4)
    palette[0] = BACK_COLOR
    palette[1] = SING_COLOR
    palette[2] = HEAD_COLOR
    palette[3] = TAIL_COLOR
    tile_grid = displayio.TileGrid(bitmap, pixel_shader=palette)
    group.append(tile_grid)

    print("Flow setup work: computing streamlines.")
    compute_streamlines()
    HEADS = [0] * len(STREAMLINES)
    print("Flow Mode resources setup complete.")
    gc.collect()

    return group

def cleanup():
    global group, bitmap, palette, tile_grid, STREAMLINES, HEADS
    print("Cleaning up Flow Mode resources...")
    if group is not None:
        while group:
            group.pop()
        group = None
    bitmap = None
    palette = None
    tile_grid = None
    STREAMLINES = []
    HEADS = []
    gc.collect()
    print("Flow Mode resources cleaned.")

def loop():
    animate_streamlines()
    update_display()
    time.sleep(DELAY)

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
