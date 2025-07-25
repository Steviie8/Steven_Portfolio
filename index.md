---
layout: default
title: Main
---

[Home](index.md) | [Code](code.md) 
# Matrix Portal Flow Visualizer
My project, the Matrix Portal Flow Visualizer, was an interesting project that had many ups and downs. Though it's called the Flow Visualizer, the main project isn't the Flow Visualizer due to the code being too glitchy and buggy; I had to pivot to another main project. This led me to the idea of different modes, where in each mode, it runs a different Python file, so it can have different modes. One of the functions of a mode is that it can access a valorant api and display the data on the screen.

```HTML You should comment out all portions of your portfolio that you have not completed yet, as well as any instructions:```
```HTML
<!--- This is an HTML comment in Markdown -->
<!--- Anything between these symbols will not render on the published site -->
```

| **Engineer** | **School** | **Area of Interest** | **Grade** |
|:--:|:--:|:--:|:--:|
| Steven X | Glenda Dawson High School | Mechanical Engineering | Incoming Junior

```HTML **Replace the BlueStamp logo below with an image of yourself and your completed project. Follow the guide [here](https://tomcam.github.io/least-github-pages/adding-images-github-pages-site.html) if you need help.**```

![Headstone Image](WIN_20250725_09_42_36_Pro.jpg)
  
# Final Milestone

**Don't forget to replace the text below with the embedding for your milestone video. Go to Youtube, click Share -> Embed, and copy and paste the code to replace what's below.**

<iframe width="560" height="315" src="https://www.youtube.com/embed/F7M7imOVGug" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>

For your final milestone, explain the outcome of your project. Key details to include are:
- In this third milestone, I have added a connection between the Matrix Portal M4 and an ESP-32 that was supposed to display data from a sensor to the screen, but due to a broken sensor, I couldn't fully display what the sensor was reading to the screen. 
- Some of the biggest challenges that I have faced are that much of the code took very long to debug, and some of them weren't the code's problem, like my sensor, which was faulty instead.
- I've learned a lot, like how to use an API and how they can integrate into code. I also learned that sometimes when solving problems, you should take it from a different angle, and sometimes you need to switch to another thing before you realize what the problem might be. 
-Something that I hope to learn in the future is the ability to integrate sensors and use the internet to integrate all these functions and be able to make a smart environment, so human tasks could be easier. This would allow people with disabilities to be able to access more functionality in their life. 
# Final Milestone Code
Here's where you'll put your code. The syntax below places it into a block of code. Follow the guide [here]([url](https://www.markdownguide.org/extended-syntax/)) to learn how to customize it to your project needs. 



# Second Milestone

<iframe width="560" height="315" src="https://www.youtube.com/embed/QSbWj-pGbiI?si=XmT6LRNO0g3uPgXG" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

For your second milestone, explain what you've worked on since your previous milestone. You can highlight:
- For the second milestone, I have added the ability to have the screen flip through different animations like a clock, a flow simulation, and finally a screen that shows my rank and how much RR I lost. This is done by fetching data from an API that works with Valorant to check my account data.
- Something that has been very surprising in my journey to make this code work was how many things that seem simple from the outside, like my Valorant API, take a long time to get working. Some of the other issues that seem surprising to me were how many difficult challenges I tried to overcome that required only 1 or 2 steps to fully fix.
- A challenge that I couldn't figure out was the ability to fetch data through an API, because CircuitPython uses a different fetch system than what normal Python works with. Though that wasn't the problem, and it was a simple, few-step fix that took me a few minutes. The problem wasn't with my code but with an outdated software that took a few clicks to fix. This has allowed to me to realize I need to think about the bigger picture on my problems, as the problem could be something that may be different then just my code.
- I need to make a network that would allow me to communicate between the screen and other devices, so I can get readings from another microcontroller and be able to display them on my screen.
  
# Second Milestone Code
[Code](code.md) 


# First Milestone

<iframe width="560" height="315" src="https://www.youtube.com/embed/6Rt9wJ12GzU?si=XAUVdCQhY9lnhJn5&amp;start=1" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

For your first milestone, describe what your project is and how you plan to build it. You can include:
- There are 2 important components in my build: the Matrix Portal M4 microcontroller and the 64x32 LED board. With these components, I will  use the microcontroller to connect to the internet and check the time and time zone of where the board is, and display it on the 64x32 LED board. 
- I've learned how to use and manipulate the software of the Matrix Portal M4 microcontroller and how to connect the board to the internet. Through my problems, I have learned how to debug the microcontroller using the serial output and other similar ways to debug the microcontroller.
- I've faced many challenges because this wasn't my original plan with the Matrix Portal M4; the original plan was to make a flow visualizer, but due to nasty code that crashed the microcontroller, I had to change the project. Even after 6 hours of trying to debugging, the instructors and I could't figure out the problem to the issue. This led me to do the Time clock.
- I could add Animations to the screen that would make the screen smoother and allow it to transition between screens. I also want to add more information from the internet to the screen, like the weather or the stock market. If I still have time, I would also like to add the score of my games on the screen.

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


# Bill of Materials

| **Part** | **Note** | **Price** | **Link** |
|:--:|:--:|:--:|:--:|
| Matrix Portal Flow Visualizer | The screen that shows LED Flow | $69.95 | <a href="https://www.adafruit.com/product/4812"> Link </a> |
| USB type A to type C cable | Connecting the Matrix Portal M4 to your computer | $3.95 | <a href="https://www.adafruit.com/product/4473"> Link </a> |
| Wire Stand | Use the stand to hold the screen in place | $4.95 | <a href="https://www.adafruit.com/product/1679"> Link </a> |
| Screwdriver | Used to put the screws into the M4 chip | $1.50 | <a href="https://www.adafruit.com/product/3284"> Link </a> |

# Other Resources/Examples
One of the best parts about Github is that you can view how other people set up their own work. Here are some past BSE portfolios that are awesome examples. You can view how they set up their portfolio, and you can view their index.md files to understand how they implemented different portfolio components.
- [Example 1](https://trashytuber.github.io/YimingJiaBlueStamp/)
- [Example 2](https://sviatil0.github.io/Sviatoslav_BSE/)
- [Example 3](https://arneshkumar.github.io/arneshbluestamp/)

To watch the BSE tutorial on how to create a portfolio, click here.
