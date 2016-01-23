# GPIO Module

This module provides access to the [GPIO](https://en.wikipedia.org/wiki/General-purpose_input/output) (General Purpose Input/Output) subsystem.

All access is based on the I/O index number on the NodeMCU dev kits, not the internal GPIO pin. For example, the D0 pin on the dev kit is mapped to the internal GPIO pin 16.

If not using a NodeMCU dev kit, please refer to the below GPIO pin maps for the index↔gpio mapping.

| IO index | ESP8266 pin | IO index | ESP8266 pin |
|---------:|:------------|---------:|:------------|
|    0 [*] | GPIO16      |        7 | GPIO13      |
|        1 | GPIO5       |        8 | GPIO15      |
|        2 | GPIO4       |        9 | GPIO3       |
|        3 | GPIO0       |       10 | GPIO1       |
|        4 | GPIO2       |       11 | GPIO9       |
|        5 | GPIO14      |       12 | GPIO10      |
|        6 | GPIO12      |          |             |

** [*] D0(GPIO16) can only be used as gpio read/write. No interrupt support. No pwm/i2c/ow support. **


## gpio.mode()

Initialize pin to GPIO mode, set the pin in/out direction, and optional internal pullup.

#### Syntax
`gpio.mode(pin, mode [, pullup])`

#### Parameters
- `pin` pin to configure, IO index
- `mode` one of gpio.OUTPUT or gpio.INPUT, or gpio.INT(interrupt mode)
- `pullup` gpio.PULLUP or gpio.FLOAT; default is gpio.FLOAT

#### Returns
`nil`

#### Example
```lua
gpio.mode(0, gpio.OUTPUT)
```
#### See also
- [`gpio.read()`](#gpioread)
- [`gpio.write()`](#gpiowrite)

## gpio.read()

Read digital GPIO pin value.

#### Syntax
`gpio.read(pin)`

#### Parameters
`pin` pin to read, IO index

#### Returns
a number, 0 = low, 1 = high

#### Example
```lua
-- read value of gpio 0.
gpio.read(0)
```
#### See also
[`gpio.mode()`](#gpiomode)

## gpio.serout()

Serialize output based on a sequence of delay-times. After each delay, the pin is toggled.

#### Syntax
`gpio.serout(pin, start_level, delay_times [, repeat_num])`

#### Parameters
- `pin`  pin to use, IO index
- `start_level` level to start on, either `gpio.HIGH` or `gpio.LOW`
- `delay_times` an array of delay times between each toggle of the gpio pin.
- `repeat_num` an optional number of times to run through the sequence.

Note that this function blocks, and as such any use of it must adhere to the SDK guidelines of time spent blocking the stack (10-100ms). Failure to do so may lead to WiFi issues or outright crashes/reboots.

#### Returns
`nil`

#### Example
```lua
gpio.mode(1,gpio.OUTPUT,gpio.PULLUP)
gpio.serout(1,1,{30,30,60,60,30,30})  -- serial one byte, b10110010
gpio.serout(1,1,{30,70},8)  -- serial 30% pwm 10k, lasts 8 cycles
gpio.serout(1,1,{3,7},8)  -- serial 30% pwm 100k, lasts 8 cycles
gpio.serout(1,1,{0,0},8)  -- serial 50% pwm as fast as possible, lasts 8 cycles
gpio.serout(1,0,{20,10,10,20,10,10,10,100}) -- sim uart one byte 0x5A at about 100kbps
gpio.serout(1,1,{8,18},8) -- serial 30% pwm 38k, lasts 8 cycles
```

## gpio.trig()

Establish a callback function to run on interrupt for a pin.

There is currently no support for unregistering the callback.

This function is not available if GPIO_INTERRUPT_ENABLE was undefined at compile time.

#### Syntax
`gpio.trig(pin, type [, function(level)])`

#### Parameters
- `pin` **1~12**, IO index, pin D0 does not support interrupt.
- `type` "up", "down", "both", "low", "high", which represent rising edge, falling edge, both edge, low level, high level trig mode correspondingly.
- `function(level)` callback function when triggered. The gpio level is the param. Use previous callback function if undefined here.

#### Returns
`nil`

#### Example

```lua
-- use pin 1 as the input pulse width counter
pin = 1
pulse1 = 0
du = 0
gpio.mode(pin,gpio.INT)
function pin1cb(level)
  du = tmr.now() - pulse1
  print(du)
  pulse1 = tmr.now()
  if level == gpio.HIGH then gpio.trig(pin, "down") else gpio.trig(pin, "up") end
end
gpio.trig(pin, "down", pin1cb)

```
#### See also
[`gpio.mode()`](#gpiomode)

## gpio.write ()

Set digital GPIO pin value.

#### Syntax
`gpio.write(pin, level)`

#### Parameters
- `pin` pin to write, IO index
- `level` `gpio.HIGH` or `gpio.LOW`

#### Returns
`nil`

#### Example
```lua
-- set pin index 1 to GPIO mode, and set the pin to high.
pin=1
gpio.mode(pin, gpio.OUTPUT)
gpio.write(pin, gpio.HIGH)
```
#### See also
- [`gpio.mode()`](#gpiomode)
- [`gpio.read()`](#gpioread)