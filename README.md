from machine import Pin, ADC
from neopixel import NeoPixel
from time import sleep

# Stepper motor control pins (ULN2003 driver)
IN1 = Pin(26, Pin.OUT)
IN2 = Pin(27, Pin.OUT)
IN3 = Pin(14, Pin.OUT)
IN4 = Pin(12, Pin.OUT)

# Stepper motor sequence (Half-step mode)
step_sequence = [
    [1, 0, 0, 0],
    [1, 1, 0, 0],
    [0, 1, 0, 0],
    [0, 1, 1, 0],
    [0, 0, 1, 0],
    [0, 0, 1, 1],
    [0, 0, 0, 1],
    [1, 0, 0, 1]
]

# Light Sensor (LDR) setup on ADC GPIO 36
ldr = ADC(Pin(36))
ldr.atten(ADC.ATTN_11DB)  # Full voltage range (0-3.3V)

# NeoPixel setup on GPIO 4 (Multiple LEDs)
NUM_PIXELS = 16  # Change this to match your number of LEDs
NEO_PIN = Pin(4, Pin.OUT)
neo = NeoPixel(NEO_PIN, NUM_PIXELS)  # Initialize all pixels

# Light detection threshold
LIGHT_THRESHOLD = 20  # Adjust based on testing

# Stepper motor movement settings
STEP_DELAY = 0.004  # Faster stepper response
STEPS_PER_REV = 512  # 1 full revolution (28BYJ-48 motor)
DEGREES_270 = int((270 / 360) * STEPS_PER_REV)  # Steps for 270°

def step_motor(steps, direction=1):
    """Rotate stepper motor by a given number of steps."""
    for _ in range(steps):
        for step in (step_sequence if direction == -1 else reversed(step_sequence)):
            IN1.value(step[0])
            IN2.value(step[1])
            IN3.value(step[2])
            IN4.value(step[3])
            sleep(STEP_DELAY)  # Faster reaction

def light_up_all(color):
    """Turn all NeoPixels to the given color."""
    for i in range(NUM_PIXELS):
        neo[i] = color  # Set each LED to the same color
    neo.write()  # Update the NeoPixels

# Main loop
while True:
    # Turn ON all NeoPixels (simulate sunlight)
    light_up_all((255, 255, 0))  # Yellow color for all LEDs
    sleep(3)  # Let LDR detect NeoPixel light

    # Read light level after NeoPixels are ON
    light_level = ldr.read()
    print(f"🌞 Light Level: {light_level}")

    if light_level > LIGHT_THRESHOLD:
        print("🔆 Light Detected! Rotating Left")
        step_motor(DEGREES_270, direction=1)  # Rotate left (CCW)
    else:
        print("🌙 No Light Detected! Rotating Right")
        step_motor(DEGREES_270, direction=-1)  # Rotate right (CW)

    # Turn OFF all NeoPixels (simulate nighttime)
    light_up_all((0, 0, 0))  # Turn off all LEDs
    sleep(8)
    # Wait before the next cycle
    if light_level > LIGHT_THRESHOLD:
        print("🔆 Light Detected! Rotating Left")
        step_motor(DEGREES_270, direction=1)  # Rotate left (CCW)
    else:
        print("🌙 No Light Detected! Rotating Right")
        step_motor(DEGREES_270, direction=-1)  # Rotate right (CW)
# Write your code here :-)
