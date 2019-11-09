# WIP MSR605X magstripe reader/writer library

This is an in progress library for the MSR605X magstripe reader/writer.  The MSR605X appears as a USB HID device and uses a protocol that appears to be a small wrapper around the older serial protocol used by other MSR
* devices.

# Protocol Details

The MSR605X uses 64 byte USB HID packets to encapsulate what appears to be the MSR605's serial protocol.  
The MSR605's serial protocol is documented in section 6 of the [MSR605 Programmer's Manual](https://usermanual.wiki/Pdf/MSR60520Programmers20Manual.325315846/help).

Messages to be sent over USB are split into chunks with a maximum size of 63 bytes. A 64 byte USB HID packet is then constructed from a 1 byte header, a chunk of the message, and sometimes some extra bytes to make the packet exactly 64 bytes regardless of the size of the chunk. The 1 byte header is made up of a single bit indicating if this packet is the first in the sequence of packets encapsulating a particular message, another single bit that indicates if this is the last packet in a sequence, and a 6 bit unsigned integer representing the length of the payload in the current packet.  For example, a header byte of 0b11000010 (0xC2) has the first packet in sequence bit (0b10000000) set, the last packet in sequence bit (0b01000000) set, and has a payload length of 0b00000010 (2).

# Encapsulation Examples

Note: while in the examples, I fill in the bytes necessary to make each packet 64 bytes long with zeroes, the actual reader and it's companion software does not and it looks like it justs sends whatever was last in memory where it's reading from (this tripped me up a bunch early on since I was trying to make sense of the whole packet).

## Single Packet Example

Message: "string"

Packet 1: 0xC6737472696E67000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000

Packet 1 header: 0b11000110 (0xC6): start of sequence, end of sequence, this packet's payload length is 6 (0b000110)

Packet 1 payload: 0x737472696E67 ("string")

## Multiple Packet Example

Message: "AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA"

Packet 1: 0xBF414141414141414141414141414141414141414141414141414141414141414141414141414141414141414141414141414141414141414141414141414141

Packet 1 header: 0b10111111 (0xBF): start of sequence, this packet's payload length is 63 (0b00111111)

Packet 2: 0x3F414141414141414141414141414141414141414141414141414141414141414141414141414141414141414141414141414141414141414141414141414141

Packet 2 header: 0b00111111 (0x3F): this packet's payload length is 63 (0b00111111)

Packet 3: 0x4F414141414141414141414141414141000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000

Packet 3 header: 0b01001111 (0x4F): end of sequence, this packet's payload length is 15 (0b001111)

# Requirements

- python 3
- pyusb from pip or another package manager (the package is often `python-pyusb` or similar)
- suitable USB backend for pyusb, see the [pyusb tutorial](https://github.com/pyusb/pyusb/blob/master/docs/tutorial.rst) for more information on backends

# Debugging Setup
- MSR605X physically attached to linux host
  - passed through to windows VM
- usbmon kernel module on host intercepts the USB packets
  - Wireshark records intercepted USB packets
- Windows VM with MSR605X gui
