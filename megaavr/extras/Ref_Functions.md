# Additional Functions in DxCore
These are in addition to what is listed in the [Arduino Reference](https://www.arduino.cc/reference/en/).

## Error reporting
One of the major challenges when writing embedded code is that there are no exceptions, so when something doesn't work correctly, you don't get an indication of what, exactly. prevented it from doing so. It's up to you to try to gather more data and analyze it to figure out where it is failing. Often, when you get to the end of it, the problem turns out to be something that one could have known ahead of time would certainly not work. While these methods are far from perfect, when we can determine at compile time that your code won't do what you are trying to make it do, will produce an error. These two "error" function calls are the mechanism used to actually create the error in almost all cases: compilation fails if any of these are referenced in the code after all constant folding and optimization.

Internally these are often used with the compiler test `__builtin_constant_p()` which returns true if the argument is a compile-time known constant value subject to constant folding:
```c
digitalWrite(100, HIGH); // This tests whether the pin is known to be invalid at compile time, and since nothing supported by this core has 100 pins, will error.
uint8_t pinnbr = 100;
digitalWrite(pinnbr, HIGH); // This will also error, because there is no chance for pinnbr to be anything other than 100.
volatile uint8_t pinnumber = 100;
digitalWrite(pinnumber, HIGH); // This will not error at compile time, because the compiler cannot optimize away the volatile variable.
```

### `void badArg("msg")`
This error means that you passed an argument to a function that is guaranteed to give garbage results, and we know at compile time, such as `analogRead(PinThatHasNoAnalogCapability)`. The message will indicate why that argument is invalid. The text will describe what was wrong with the argunment.

### `void badCall("msg")`
This error means that you called a function that does not make sense to call with the current hardware or tools submenu selections, regardless of what values one passes. For example, calling `stop_millis()` when millis is disabled. The message will indicate why that call is invalid.

## Digital Functions
See [Digital Reference](https://github.com/SpenceKonde/DxCore/blob/master/megaavr/extras/Ref_Digital.md)
```c
int8_t  digitalReadFast( uint8_t pin)
void    digitalWriteFast(uint8_t pin, uint8_t val)
void    openDrainFast(   uint8_t pin, uint8_t val)
void    openDrain(       uint8_t pin, uint8_t val)
void    pinConfigure(    uint8_t pin, uint16_t mode)
void    turnOffPWM(      uint8_t pin)
```
## Pin information
These are almost all preprocessor macros, not functions, but what they expand to is appropriate for the stated datatypes/
These are in many cases part of the standard API, but not all are. The concept of analog channel identifiers (eg, ADC_CH(0) = 0x80) is new in my cores, and gives an unambiguous way to represent an input by either pin or channel. Historically, this has been a mess on non-328p parts. Note that *digital I/O functions do not accept channel identifiers at this time on DxCore* - I am trying to train people away from that practice.

As a reminder, we recommend getting in the habit of using PIN_Pxn notation rather than raw numbers - it makes your code more portable between modern AVR parts.


### `uint8_t digitalPinToPort(pin)`
(standard) Returns the port that a pin belongs to - these are constants named PA, PB, PC, etc (not to be confused with PORTA, which is the port struct itself). These have numeric values of 0-6 for the existing poerrts, PA to PG.

### `uint8_t digitalPinToBitPosition(pin)`
(standard) Returns the bit position of a pin within a port, eg, `digitalPinToBitPosition(PIN_PA5)` will return 5.

### `uint8_t digitalPinToBitMask(pin)`
(standard) Returns the bit mask of a pin within a port, eg, `digitalPinToBitPosition(PIN_PA5)` will return (1 << 5) or 0b00100000

### `bool digitalPinHasPWM(pin)`
Returns true if the pin - at default core configuration - has PWM. Does not account for user changed to PORTMUX, but the dynamic nature of `digitalPinToTimerNow()` causes problems in some cases.

### `uint8_t digitalPinToTimer(pin)`
Returns the timer used for PWM - at default core configuration, assuming the pin has PWM. Does not account for user changes to PORTMUX, but the dynamic nature of `digitalPinToTimerNow()` causes problems in some cases.

### `PORT_t* portToPortStruct(port)`
### `PORT_t* digitalPinToPortStruct(pin)`
(standard) These get a pointer to the port structure associated with either the given port or the port that the pin is a member of

### `volatile uint8_t * getPINnCTRLregister(port, bit_pos)`
Returns a pointer to the PINnCTRL register associated with that port and bit position.

### `volatile uint8_t * portOutputRegister(P)`
### `volatile uint8_t * portInputRegister(P)`
### `volatile uint8_t * portModeRegister(P)`
These return pointers to the port output, input and direction registers (output controls whether the pin is driven high or low if driven, direction controls whether the pin is driven or is an input, and input is the read-only register containing the input value).

### `uint8_t digitalPinToAnalogInput(p)`
(standard) Returns the analog input number used internally. Only useful when fully taking over the ADC either directly or as part of a sketch.

### `uint8_t analogChannelToDigitalPin(p)`
Returns the pin number corresponding to an analog channel identifier. This is simply the analog input number with the high bit set via the ADC_CH() macro. Think long and hard if you find yourself needing this, it is rarely important on DxCore since you can always refer to pins with the digital pin number or even better, PxN notation.

### `uint8_t analogInputToDigitalPin(p)`
(standard) Returns the digital pin number associated with an analog input number. Only useful when fully taking over the ADC either directly or as part of a sketch.

### `uint8_t digitalOrAnalogPinToDigital(p)`
If `p < NUM_DIGITAL_PINS`, p is a digital pin, and is returned as is. If it is equal to `ADC_CH(n)` where n is a valid analog channel it is converted to the digital pin, and if anything else, it returns NOT_A_PIN.

### `uint8_t portToPinZero(port)`
Returns `PIN_Px0` - eg `portToPinZero(PA)` is PIN_PA0. This will not work if there is no "hole" provided in the pin numbering.
For example, AVR64DD14, the pins are PA0, PA1, PC1, PC2, PC3, PD4, PD5, PD6, PD7, PF6, PF7, with numbers 0, 1, 2, 3, 4, ~5 (PD0), 6, 7, 8~ 9, 10, 11, 12, 13, 14.  portToPinZero will work for PA and PD, but not PC or PF; this function is used internally in ways that don't matter for this case, and because it would otherwise involve the same pin number meaning different things, or much larger arrays (hence wasted space)

### `volatile uint8_t* digitalPinHasPWMTCB(p)`

(Standard) Returns true if the pin has PWM available in the standard core configuration. This is a compile-time-known constant as long as the pin is, and does not account for the PORTMUX registers.
### `uint8_t digitalPinHasPWM(p)`

(Standard) Returns true if the pin has PWM available in the standard core configuration. This is a compile-time-known constant as long as the pin is, and does not account for the PORTMUX registers.

## Attach Interrupt Enable
If using the old or default (all ports) options, these functions are not available; WInterrupt will define ALL port pin interrupt vectors if `attachInterrupt()` is referemced amywhere in the sketch or included library. This is the old behavior.
If you are using the new implementation in manual mode you must call one of the following functions before attaching the interrupt (override `onPreMain()` if you need it ready for a class constructor. That port can have interrupts attached, but others cannot. That port cannot have manuallly written interrupts, which are typically 10-20 times faster), while the other ports can. I will also note that the implementation is hand-tuned assembly and that's still how slow it is. See the [Interrupt Reference for more information](https://github.com/SpenceKonde/DxCore/blob/master/megaavr/extras/Ref_Interrupts.md).
```c
void attachPortAEnable()
void attachPortBEnable()
void attachPortCEnable()
void attachPortDEnable()
void attachPortEEnable()
void attachPortFEnable()
void attachPortGEnable()
```
## Overridables
See [Callback Reference](https://github.com/SpenceKonde/DxCore/blob/master/megaavr/extras/Ref_Callbacks.md)
```c++
  void init_reset_flags()
  void onPreMain()
  void onBeforeInit()
  void init() // Part of ArduinoAPI
  void initVariant() //part of ArduinoAPI - reserved for core and rare libraries
  void init_TCA0()
  void init_TCA1()
  void init_TCBs()
  void init_TCD0()
  void init_millis()
  uint8_t onAfterInit()
  void onClockFailure()
  void onClockTimeout()
  int main(); // the Big One!
```

### `Related: uint8_t digitalPinToInterrupt(P)`
This is an obsolete macro that is only present for compatibility with old code. It has nothing do do and simply expands to the sole arguent.


## Analog Functions
See [Analog Reference](https://github.com/SpenceKonde/DxCore/blob/master/megaavr/extras/Ref_Analog.md)
```c
  int32_t analogReadEnh( uint8_t pin,              uint8_t res, uint8_t gain)
  int32_t analogReadDiff(uint8_t pos, uint8_t neg, uint8_t res, uint8_t gain)
  int16_t analogClockSpeed(int16_t frequency,      uint8_t options)
  bool    analogReadResolution(uint8_t res)
  bool    analogSampleDuration(uint8_t dur)
  void    DACReference(        uint8_t mode)
  uint8_t getAnalogReference()
  uint8_t getDACReference()
  uint8_t getAnalogSampleDuration()
  int8_t  getAnalogReadResolution()
```

## Timekeeping

### `delay()`
(standard) This works normally. If millis is disabled, the builtin avrlibc implementation is `_delay_ms()` is used: for constant arguments it is used directly andfor variable ones it is called in a loop with 1ms delay. The catch with that is that if interrupts fire in the middle, timespent in those does not count towards the delay.

### `delayMicroseconds()`
(standard) If called with a constant argument (which you should strive to always do) it will use the highly accurate builtin `_delay_us()` from avrlibc. Otherwise, it will use an Arduino-style loop, but with protection against inaccurate timing of short delays.

### `uint16_t clockCyclesPerMicrosecond()`
Part of the standard API, but not documented. Does exactly what it says.
```c
inline uint16_t clockCyclesPerMicrosecond() {
  return ((F_CPU) / 1000000L);
}
```

### `uint32_t clockCyclesToMicroseconds(uint32_t cycles)`
Part of the standard API, but not documented. Does exactly what the name implies. Note that it always rounds down, like everything in C.
```c
inline unsigned long clockCyclesToMicroseconds(unsigned long cycles) {
  return (cycles / clockCyclesPerMicrosecond());
}
```

### `uint32_t microsecondsToClockCycles(uint32_t microseconds)`
Part of the standard API, but not documented. Does exactly what the name implies. Note that it always rounds down, like everything in C.
```c
inline unsigned long microsecondsToClockCycles(unsigned long microseconds) {
  return (microseconds * clockCyclesPerMicrosecond());
}
```

### `_NOP()`
(standard) Execute a single cycle NOP (no operation) instruction which takes up 1 word of flash.

### `_NOPNOP()` or `_NOP2()`
Execute a 2 cycle NOP (no operation) instruction (`rjmp .+0`)which takes up 1 word of flash. (Added 1.3.9)

### `_NOP8()`
Execute an 8 cycle NOP (no operation) instruction sequence (`rjmp .+2 ret rcall .-4`) which takes up 3 words of flash. Saves 1 word vs doing via loop like below (Added 1.3.9)

### `_NOP14()`
Execute an 14 cycle NOP (no operation) instruction sequence (`rjmp .+2 ret rcall .-4 rcall .-6`) which takes up 4 words of flash, 1 word less vs doing via loop like below. (Added 1.3.9)

Longer clock-counting delays aremore efficiently done with the 3 cycle loop (ldi, dec, brne .-4) which gives 3 clocks per iteration plus 1 for the ldi. Can then be padded with `_NOP()` or `_NOPNOP()`for numbers of cycles equal to 3 x iterations + 1 in 4 words, and 3 x iterations + 2 or 3 in 5 words

## Time Manipulation

### `void stop_millis()`
Stop the timer being used for millis, and disable the interrupt. This does not change the current count. Intended for internal use in the future timing and sleep library, so you would stop it before sleeping and start the RTC before going into standby sleep, or powerdown with the PIT.

### `void set_millis(uint32_t newmillis)`
Sets the millisecond timer to the specified number of milliseconds. Be careful if you are setting to a number lower than the current millis count if you have any timeouts ongoing, since stanard best practice is to always subtract `(oldmillis - millis())` and these are unsigned. So setting it oldmillis-1 will make it look like 4.2xx billion ms have passed. and the timer will expire.

### `void restart_millis()`
After having stopped millis either for sleep or to use timer for something else and optionally have set it to correct for passage of time, call this to restart it.

### `void nudge_millis(uint16_t ms)`
This is not yet implemented as we assess whether it is a useful or appropriate addition, and how it fits in with set millis().
~Sets the millisecond timer forward by the specified number of milliseconds. Currently only implemented for TCB, TCA implementation will be added in a future release. This allows a clean way to advance the timer without needing to do the work of reading the current value, adding, and passing to `set_millis()`  It is intended for use before  (added becauise *I* needed it, but simple enough).
The intended use case is when you know you're disabling interrupts for a long time (milliseconds), and know exactly how long that is (ex, to update neopixels), and want to nudge the timer
forward by that much to compensate. That's what *I* wanted it for.~

### `_switchInternalToF_CPU()`
Call this if you are running from the internal clock, but it is not at F_CPU - likely when overriding `onClockTimeout()`  `onClockFailure()` is generally useless.

## PWM control
See [Timer Reference](https://github.com/SpenceKonde/DxCore/blob/master/megaavr/extras/Ref_Timers.md)
```text
  void takeOverTCA0()
  void takeOverTCA1()
  void takeOverTCD0()
  void resumeTCA0()
  void resumeTCA1()
  bool digitalPinHasPWMNow(uint8_t p)
  uint8_t digitalPinToTimerNow(uint8_t p)
```
