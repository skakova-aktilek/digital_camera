import sensor
import display
import time
import os
from pyb import Pin, LED, RTC



button = Pin("P6", Pin.IN, Pin.PULL_UP)
led = LED(1)
rtc = RTC()

# Set date/time manually
# Format: (year, month, day, weekday, hour, minute, second, subseconds)
# Weekday can be 1-7
rtc.datetime((2026, 7, 9, 4, 1, 55, 0, 0))  # CHANGE THIS

# Camera setup
sensor.reset()
sensor.set_pixformat(sensor.RGB565)

# Stable quality/resolution
sensor.set_framesize(sensor.QQVGA2)  # 128x160
sensor.skip_frames(time=2000)

# LCD setup
lcd = display.SPIDisplay()

# Find next photo number
counter = 0
while True:
    filename = "/sdcard/photo_%03d.jpg" % counter
    try:
        os.stat(filename)
        counter += 1
    except OSError:
        break

status = ""
status_until = 0
last_lcd_update = time.ticks_ms()

def get_timestamp():
    dt = rtc.datetime()
    return "%02d/%02d/%02d %02d:%02d:%02d" % (
        dt[1],      # month
        dt[2],      # day
        dt[0] % 100,# year
        dt[4],      # hour
        dt[5],      # minute
        dt[6]       # second
    )

while True:
    img = sensor.snapshot()

    # Button pressed
    if button.value() == 0:
        time.sleep_ms(80)  # debounce

        if button.value() == 0:
            status = "SAVING"

            filename = "/sdcard/photo_%03d.jpg" % counter

            try:
                # Save clean photo BEFORE drawing LCD text
                img.save(filename, quality=80)
                counter += 1

                status = "SAVED"
                status_until = time.ticks_add(time.ticks_ms(), 1000)

                # LED blink = saved
                led.on()
                time.sleep_ms(800)
                led.off()

                # Let SD card finish writing
                time.sleep_ms(800)

            except Exception as e:
                status = "ERROR"
                status_until = time.ticks_add(time.ticks_ms(), 1500)

                # Fast blink = error
                for i in range(4):
                    led.on()
                    time.sleep_ms(100)
                    led.off()
                    time.sleep_ms(100)

            # Wait until button released
            while button.value() == 0:
                time.sleep_ms(30)

    # Clear status after time
    if status and time.ticks_diff(status_until, time.ticks_ms()) <= 0:
        status = ""

    # LCD update
    if time.ticks_diff(time.ticks_ms(), last_lcd_update) > 100:
        img.draw_string((2, 2), "PHOTO:%03d" % counter)
        img.draw_string((2, 14), get_timestamp())

        if status:
            img.draw_string((2, 26), status)

        lcd.write(img)
        last_lcd_update = time.ticks_ms()

    time.sleep_ms(5)
