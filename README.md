# MicroPython YS-RF34T 433MHz ASK/OOK UART Transceiver

MicroPython examples using YS-RF34T 433MHz ASK/OOK UART transceivers.

This module consists of an unlabelled MCU that provides a UART interface to two daughter ASK/OOK modules.

A 433MHz receiver (YS-RF470) and a 433MHz transmitter (H34C).

This module can be used to emulate or program a ASK/OOK based fixed code remotes.

Does not support rolling code.

# Examples

```python
from machine import Pin, UART
uart = UART(1, tx=14, rx=15, baudrate=9600)

# sniff for a remote
def sniff():
    while True:
        if uart.any():
            print(uart.readline())

# eg. received b'\xfd\x84\x19\x04D\xdf'

# replay a remote
def replay():
    # almost the same buf as received
    # byte 1 is how long to tx for
    uart.write(bytearray(b'\xfd\x03\x84\x19\x04D\xdf'))

# transmit
def tx(addr, key, time=3, width=64):
	buf = bytearray(b'\xfd\x00\x00\x00\x00\x00\xdf')
	buf[1] = time & 0xff
	buf[2] = (addr >> 8) & 0xff
	buf[3] = addr & 0xff
	buf[4] = width & 0xff
	uart.write(buf)

# (0x84 << 8) | 0x19 = 33817
tx(33817, 4)

# receive
def rx():
    while True:
        if uart.any():
            buf = uart.readline()
            if len(buf) == 6 and buf[0] == 0xfd and buf[5] == 0xdf:
                addr = (buf[1] << 8) | buf[2]
                key = buf[3]
                print('Found! Address: {}, Key: {}'.format(addr,key))

rx()
# Found! Address: 33817, Key: 1
# Found! Address: 33817, Key: 2
# Found! Address: 33817, Key: 4
# Found! Address: 33817, Key: 8
```

The MCU understands 2262, 2260, 1527 encoded transmissions.

If you're not receiving anything, check your RX and TX are not switched otherwise the remote might be using an unsupported protocol.

## Pinout

Pin | Name | Description
:--:|:----:|:--------------------------------
1   | GND  | Ground
2   | RXD  | UART receive
3   | TXD  | UART transmit
4   | VCC  | Supply voltage 3V3 - 5V

## Connections

### TinyPICO ESP32

```python
from machine import Pin, UART
uart = UART(1, tx=14, rx=15, baudrate=9600)
```

YS-RF34T | TinyPICO (ESP32)
-------- | ----------------
GND      | GND
RXD      | 14 TXD
TXD      | 15 RXD
VCC      | 3V3

## Transmit Protocol

Write 7 bytes and the MCU will build and send out the desired packet using the onboard ASK/OOK RF module.

Packet = (Header, Time, Address 2, Address 1, Key, Width, Footer), eg.

byte | name   | description
---- | ------ | -----------
0    | header | constant value of 0xFD
1    | time   | transmit time - signal repeats until timeout
2    | addr2  | high address bits
3    | addr1  | low address bits
4    | key    | which button was pressed
5    | width  | width of signal in ms
6    | footer | constant value of 0xDF


header | time | addr2 | addr1 | key  | width | footer | eg
------ | ---- | ----- | ----- | ---- | ----- | ------ | -------------------------------
0xFD   | 0x03 | 0x84  | 0x19  | 0x04 | 0x40  | 0xDF   | b'\xfd\x03\x84\x19\x04\x40\xdf'
0xFD   | 0x06 | 0x22  | 0x33  | 0x44 | 0x50  | 0xDF   | b'\xfd\x06\x22\x33\x44\x50\xdf'

# Receive Protocol

When the MCU receives a signal from the receiving ASK/OOK RF module, it will output 6 bytes over UART.

The packet is almost identical to the TX packet, only is missing the Time value.

byte | name   | description
---- | ------ | ------------------------
0    | header | constant value of 0xFD
1    | addr2  | high address bits
2    | addr1  | low address bits
3    | key    | which button was pressed
4    | width  | width of signal in ms
5    | footer | constant value of 0xDF

header | addr2 | addr1 | key  | width | footer | eg
------ | ----- | ----- | ---- | ----- | ------ | ---------------------------
0xFD   | 0x84  | 0x19  | 0x04 | 0x44  | 0xDF   | b'\xfd\x84\x19\x04\x44\xdf'


## RTL-SDR capture with URH and replay with YS-RF34T

See [RTL-SDR-URH.md](RTL-SDR-URH.md).

## Parts

* [YS-RF34T](https://www.aliexpress.com/item/32963111334.html) $5.85 AUD
* [TinyPICO](https://www.tinypico.com/) $20.00 USD
* [RTL-SDR](https://www.rtl-sdr.com/buy-rtl-sdr-dvb-t-dongles/) $21.95 AUD

## Links

* [micropython.org](http://micropython.org)
* [TinyPICO Getting Started](https://www.tinypico.com/gettingstarted)
* [URH](https://github.com/jopohl/urh)
* [RTL-SDR](https://www.rtl-sdr.com)

## License

Licensed under the [MIT License](http://opensource.org/licenses/MIT).

Copyright (c) 2019 Mike Causer
