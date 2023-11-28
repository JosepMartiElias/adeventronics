# Pinout

![pinout SVG](../images/Pinout.svg)

## Inputs

There are some available inputs on the board that uses different pins. 

- Tactile push button: Connected to **GPIO00**.

!!! warning
    GPIO00 is used to start Boot mode if it's pressed while powering/reseting the board, when the board is already on, you can program it to read it as an input. 

- Capacitive pins: Pins **TOUCH01**, **TOUCH02**, **TOUCH04**, **TOUCH05** and **TOUCH06** are available with a capacitive pad on the board. 

!!! bug
    Pin TOUCH03 is also exposed, but it cannot be used as it is used by the Phototransitor circuit. 

- Phototransistor: Connected to **GPIO03**.

- Temperature Sensor: TMP102 connected by I^2^C by **GPIO08** and **GPIO09**.  

## Outputs

There are some available outputs on the board that uses different pins. 

- LED: Connected to **GPIO48**.
- Buzzer: Connected to **GPIO14**.
- Neopixel: Connected to **GPIO38**.

!!! note
    The Data Out pin of the Neopixel is connected to the DO pin, to continue the Neopixel strip from there. 

