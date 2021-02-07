# machine.Pin

The `machine` module is quite big and has a few submodules, so I'm going to treat each of those to a separate page of documentation.

The `machine.Pin` module is for basic control of the input/output (I/O) pins on the Pico.  It will allow you to light LEDs, signal driver chips to turn motors, and detect input signals from buttons and more.  This module also allows the setup of pin interrupts, a way of signalling the software to perform a task as soon as a button changes.

## `machine.Pin(pinNum, mode, pull, value)`

The `machine.Pin()` function is the basic function in this module, and allows you to define an object to control an I/O pin.  Only the `pinNum` parameter is required.

* The `pinNum` parameter should be the number of the pin to use (this is the GP number from the [official pinout diagram](https://datasheets.raspberrypi.org/pico/Pico-R3-A4-Pinout.pdf).  Note: you only need the number, don't include the "GP" part).
* The `mode` parameter should be `Pin.OUT`, `Pin.IN`, `Pin.OPEN_DRAIN`, or `Pin.ALT`.  `Pin.OUT` sets the pin to be an output pin, and `Pin.IN` sets it to be an input pin.  `Pin.ALT`, according to the official MicroPython docs, configures the pin for "an alternative function".  I assume this is something to do with using functions like I2C and SPI, but the official docs. for those don't mention this function, so maybe it isn't really useful?  The `Pin.OPEN_DRAIN` seems a little cryptic to me, the MicroPython docs state that in this mode if the pin is set to 0 the pin is "active at a low level", and if set to 1 is "in a high impedance state".  The distinction here seems to be a subtle electronics thing.
* The `pull` parameter allows you to set a pull-up (`Pin.PULL_UP`) or pull-down (`Pin.PULL_DOWN`) resistor on a pin.  This controls the initial state of input pins and also some electrical behaviour which is beyond the scope of this document.
* The `value` parameter sets the initial state of pins which are using the `Pin.OUT` or `Pin.OPEN_DRAIN` modes.  Use `1` to start the pin in a hight state, or `0` to start it low.

Use these to set a pin up for use in your program:

```python
from machine import Pin

led = Pin(16, Pin.OUT, 1)               # Set up pin 16 as an output
button = Pin(17, Pin.IN, Pin.PULL_DOWN) # Set up pin 17 as an input with a pull-down resistor
```

## `Pin.init(mode, pull, value, alt)`

Reinitialises the pin with the given parameters (see the main `machine.Pin` section for valid values).  I'm not clear on the valid values for `alt`, although it accepts `True` and `False`.

```python
from machine import Pin

testPin = Pin(16, Pin.IN, Pin.PULL_DOWN)      # Set pin 16 to be an input called testPin
testPin.init(Pin.OUT,Pin.PULL_DOWN, value=1)  # Reinitialise testPin as an output

```

## `Pin.deinit()`

This function is present in the MicroPython port files, but doesn't seem to work.  The code implies that this is about resetting all pins and removing and interrupts attached to them.


## `Pin.print`

This function seems to run whenever you call the line `print(pinObject)`.  This will tell you which physical pin a `Pin` object is attached to, which mode it is running in, and will also tell you whether the `Pin` has a pull-up or -down LED applied.

```python
from machine import Pin

testPin = Pin(16, Pin.IN, Pin.PULL_DOWN)
print(testPin)    # Outputs `Pin(16, mode=IN, pull=PULL_DOWN)`
```

## `Pin.call`

Not a function itself, but a shortcut to setting the value of output pins or getting the value of input pins.  Run `pinObject()` to read an input, or `pinObject(value)` to set an output pin to the given value (`True`/`1` or `False`/`0`).

```python
from machine import Pin

led = Pin(16, Pin.OUT, Pin.PULL_DOWN)
led(1)  # Sets the pin to high
led(0)  # Sets the pin to low

button = Pin(17, Pin.IN, Pin.PULL_DOWN)
is_pressed = button()
print(is_pressed) # Prints '1' if the button is pressed, or '0' if not.
```

## `Pin.value()`

Use this to set the output state of a pin.  It works in the same way as `Pin.call` but makes your code a bit more explicit when read.

```python
from machine import Pin

led = Pin(16, Pin.OUT, Pin.PULL_DOWN)
led.value(1)  # Set the led pin to high
led.value(0)  # Set the led pin to low
```

## `Pin.high()` and `Pin.low()`

Again, use these to set the state of output pins.

```python
from machine import Pin

led = Pin(16, Pin.OUT, Pin.PULL_DOWN)
led.high()  # Set the led pin to high
led.low()   # Set the led pin to low
```

## `Pin.toggle()`

Sets the output state of a pin to the opposite of its current state.

```python
from machine import Pin

led = Pin(16, Pin.OUT, Pin.PULL_DOWN)
led.toggle()  # Toggles the state of the pin
```

## `Pin.irq(handler, trigger, hard)`

Use this to set up the pin as an interrupt, i.e. when the appropriate change on the pin's input is detected your code should immediately jump to the function set as the `handler`.

* `handler`: the function to call when the interrupt is triggered.  This function must take only one parameter, and when the interrupt is triggered then the pin object the interrupt is attached to is passed through that parameter 
* `trigger`: Should be `Pin.IRQ_FALLING` (the interrupts is triggered when the input drops form high to low) or `Pin.IRQ_RISING` (the interrupts is triggered when the input rises from low to high).  These can be `or`-ed together to detect either change.
* `hard` apparently, according to the MicroPython docs, sets whether or not to use a hardware interrupt, which may be faster to trigger.  I'm not sure precisely what diference this makes in reality.

```python
from machine import Pin
import time

def handleTheInterrupt(pin):
    print(pin)  # Will print 'Pin(17, mode=IN, pull=PULL_DOWN)'

led = Pin(17, Pin.IN, Pin.PULL_DOWN)
led.irq(handleTheInterrupt, Pin.IRQ_FALLING or Pin.IRQ_RISING, hard=0)

while True:
    pass
```
