# CS-350 Smart Thermostat (Raspberry Pi 4)

A working smart thermostat prototype built for SNHU **CS-350 Emerging Systems Architectures and Technologies**. It runs on a Raspberry Pi 4B in Python 3 and brings together every peripheral covered in the course (GPIO, PWM, interrupts, I2C, and UART), coordinated by a state machine.

## What it does

- **Three modes: Off, Heat, Cool.** A mode button cycles through them.
- **Adjustable setpoint.** Up and down buttons move the setpoint 1°F per press (default 72°F, limited to 50°F through 90°F).
- **Temperature sensing.** An AHT20 sensor on the I2C bus is read once per second.
- **Status lights.** The red LED (heat) or blue LED (cool) fades while the system is calling for heating or cooling and goes solid once the setpoint is met. Both are dark in Off.
- **16x2 LCD.** Line 1 shows the date and time. Line 2 alternates every 5 seconds between the room temperature and the current mode with its setpoint.
- **Server updates.** Every 30 seconds the thermostat sends a comma-delimited status line over UART (115200 baud, 8N1), for example `heat,71.6,72`. This stands in for the cloud connection planned for the next phase.

## Hardware

| Component | Interface | Raspberry Pi pins |
|---|---|---|
| AHT20 temperature and humidity sensor | I2C (address 0x38) | GPIO 2 (SDA), GPIO 3 (SCL) |
| 16x2 character LCD | 4-bit parallel | RS 17, EN 27, D4 5, D5 6, D6 13, D7 26 |
| Red LED (heat) | PWM | GPIO 18 |
| Blue LED (cool) | PWM | GPIO 23 |
| Mode button | GPIO input, interrupt | GPIO 24 |
| Setpoint up button | GPIO input, interrupt | GPIO 25 |
| Setpoint down button | GPIO input, interrupt | GPIO 12 |
| USB-to-TTL serial cable | UART (`/dev/ttyS0`) | GPIO 14 (TXD), GPIO 15 (RXD) |

Buttons use 10k pull-up resistors on the breadboard. A potentiometer sets the LCD contrast.

**Wiring map:** [CS350_Breadboard_Map.xlsx](CS350_Breadboard_Map.xlsx) documents the breadboard at every stage of the build, from the Module One LED to the finished thermostat. Each tab has a top-down hole-by-hole layout and a connections table, plus a blank template for mapping new stages.

## Software

- Raspberry Pi 4B running Ubuntu Server 24.04, managed headless over SSH
- Python 3 with `gpiozero` (buttons and PWM LEDs), `python-statemachine`, Adafruit Blinka with the CircuitPython AHTx0 and CharLCD libraries, and `pyserial`

## Design highlights

- **Interrupt-driven input.** Button callbacks fire on the GPIO edge, so the program never polls the buttons. A 50 ms debounce keeps one press to one step.
- **One reading, many consumers.** A single sensor read each second feeds the LCD, the LEDs, and the UART, so all three always agree.
- **Steady timing.** The display loop holds a fixed 1 second clock, so the 30 second server update does not drift.
- **No flicker on refresh.** An LED only changes when its pattern changes, so the periodic light refresh never restarts a fade that is already running.
- **Fails safe.** A failed I2C read shows "Sensor error" on the LCD instead of stopping the display thread.
- **Clean shutdown.** Ctrl+C turns off the LEDs, clears and releases the LCD, and closes the serial port.

## Next phase: Wi-Fi architecture

For a production version with built-in Wi-Fi, I compared the Raspberry Pi Zero 2 W, the Microchip PIC32MZ W1, and the NXP RW612 against three requirements: peripheral support, cloud connectivity, and memory. I recommended the **Raspberry Pi Zero 2 W** because it keeps the same 40-pin header and runs the existing Python code unchanged. Both microcontrollers would need a full rewrite in C.

## Repository status

Source code, the state machine diagram, and the architecture report are being added next, along with a reflection on the project.

## Author

Eric Shadwick, B.S. Computer Science (Software Engineering) student, Southern New Hampshire University
