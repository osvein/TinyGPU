# TinyGPU
High-level graphics processing unit for KS0108-based LCDs

## Instruction set
The 0th byte of an instruction consists of 8 bits.
The 7 most significant bits identify the instruction.
The least significant bit indicates whether it's a read or write operation (1 = read, 0 = write).

| Name         | 0th byte  |
| ------------ | --------- |
| Status       | 0000 000x |
| Colour logic | 0000 001x |
| Start line   | 0000 010x |
| Pixel        | 0000 100x |
| Polygon      | 0000 110x |
| Polyellipse  | 0000 111x |

The exact behaviour of each instruction, including the difference between read and write operations, is explained below.

### Status
**1st byte:**

|          | 7  MSB | 6      | 5      | 4      | 3      | 2      | 1      | 0  LSB |
| -------- | ------ | ------ | ------ | ------ | ------ | ------ | ------ | ------ |
| MOSI (w) |        |        |        |        |        |        | on/off | reset  |
| MISO (w) |        |        |        |        |        |        |        |        |
| MOSI (r) |        |        |        |        |        |        |        |        |
| MISO (r) |        |        |        |        |        |        | on/off | idle   |

*reset:*
- 1 = soft reset GPU
    - LCD will be hard reset when the GPU is reset
- 0 = don't reset

*on/off:*
- 1 = connect LCD
    - has no effect if already connected
- 0 = disconnect LCD
    - has no effect if already disconnected
    - does not affect display data (i.e. will be redrawn when reconnected)

*idle:*
- 1 = idle - currently not processing and no operations are pending
- 0 = processing

### Colour logic
**1st byte:**

|          | 7  MSB | 6      | 5      | 4      | 3      | 2      | 1      | 0  LSB |
| -------- | ------ | ------ | ------ | ------ | ------ | ------ | ------ | ------ |
| MOSI (w) |        |        |        |        |        |        | f(1)   | f(0)   |
| MISO (w) |        |        |        |        |        |        |        |        |
| MOSI (r) |        |        |        |        |        |        |        |        |
| MISO (r) |        |        |        |        |        |        | f(1)   | f(0)   |

*f(0):* the resulting colour if the old colour is black

*f(1):* the resulting colour if the old colour is white

### Start line
**1st byte:**

|          | 7  MSB | 6      | 5      | 4      | 3      | 2      | 1      | 0  LSB |
| -------- | ------ | ------ | ------ | ------ | ------ | ------ | ------ | ------ |
| MOSI (w) |        |        | line   | line   | line   | line   | line   | line   |
| MISO (w) |        |        |        |        |        |        |        |        |
| MOSI (r) |        |        |        |        |        |        |        |        |
| MISO (r) |        |        | line   | line   | line   | line   | line   | line   |

*line:* the line at the top of the display

### Pixel
**1st byte:**

|          | 7  MSB | 6      | 5      | 4      | 3      | 2      | 1      | 0  LSB |
| -------- | ------ | ------ | ------ | ------ | ------ | ------ | ------ | ------ |
| MOSI (w) | count  | count  | count  | count  | count  | count  | count  | count  |
| MISO (w) |        |        |        |        |        |        |        |        |
| MOSI (r) | count  | count  | count  | count  | count  | count  | count  | count  |
| MISO (r) |        |        |        |        |        |        |        |        |

**2nd byte:**

|          | 7  MSB | 6      | 5      | 4      | 3      | 2      | 1      | 0  LSB |
| -------- | ------ | ------ | ------ | ------ | ------ | ------ | ------ | ------ |
| MOSI (w) |        | x      | x      | x      | x      | x      | x      | x      |
| MISO (w) |        |        |        |        |        |        |        |        |
| MOSI (r) |        | x      | x      | x      | x      | x      | x      | x      |
| MISO (r) |        |        |        |        |        |        |        |        |

**3rd byte:**

|          | 7  MSB | 6      | 5      | 4      | 3      | 2      | 1      | 0  LSB |
| -------- | ------ | ------ | ------ | ------ | ------ | ------ | ------ | ------ |
| MOSI (w) |        |        | y      | y      | y      | y      | y      | y      |
| MISO (w) |        |        |        |        |        |        |        |        |
| MOSI (r) |        |        | y      | y      | y      | y      | y      | y      |
| MISO (r) | value  |        |        |        |        |        |        |        |

**The 2nd and 3rd bytes are repeated *count* times in unsorted order. (i.e. *2nd-3rd-2nd-3rd-2nd-3rd* for *count = 3*.)**

*count:* the number of pixels

*x:* the x (horizontal) coordinate of the pixel

*y:* the y (vertical) coordinate of the pixel

*value:* the value of the pixel

### Polygon
**1st byte:**

|          | 7  MSB | 6      | 5      | 4      | 3      | 2      | 1      | 0  LSB |
| -------- | ------ | ------ | ------ | ------ | ------ | ------ | ------ | ------ |
| MOSI (w) | count  | count  | count  | count  | count  | count  | count  | count  |
| MISO (w) |        |        |        |        |        |        |        |        |

**2nd byte:**

|          | 7  MSB | 6      | 5      | 4      | 3      | 2      | 1      | 0  LSB |
| -------- | ------ | ------ | ------ | ------ | ------ | ------ | ------ | ------ |
| MOSI (w) |        | x      | x      | x      | x      | x      | x      | x      |
| MISO (w) |        |        |        |        |        |        |        |        |

**3rd byte:**

|          | 7  MSB | 6      | 5      | 4      | 3      | 2      | 1      | 0  LSB |
| -------- | ------ | ------ | ------ | ------ | ------ | ------ | ------ | ------ |
| MOSI (w) |        |        | y      | y      | y      | y      | y      | y      |
| MISO (w) |        |        |        |        |        |        |        |        |

**The 2nd and 3rd bytes are repeated *count* times in unsorted order. (i.e. *2nd-3rd-2nd-3rd-2nd-3rd* for *count = 3*.)**

**The behaviour of reading is undefined.**

*count:* the number of points

*x:* the x (horizontal) coordinate of the point

*y:* the y (vertical) coordinate of the point

### Polyellipse
**1st byte:**

|          | 7  MSB | 6      | 5      | 4      | 3      | 2      | 1      | 0  LSB |
| -------- | ------ | ------ | ------ | ------ | ------ | ------ | ------ | ------ |
| MOSI (w) | radius | radius | radius | radius | radius | radius | radius | radius |
| MISO (w) |        |        |        |        |        |        |        |        |

**2nd byte:**

|          | 7  MSB | 6      | 5      | 4      | 3      | 2      | 1      | 0  LSB |
| -------- | ------ | ------ | ------ | ------ | ------ | ------ | ------ | ------ |
| MOSI (w) | count  | count  | count  | count  | count  | count  | count  | count  |
| MISO (w) |        |        |        |        |        |        |        |        |

**3rd byte:**

|          | 7  MSB | 6      | 5      | 4      | 3      | 2      | 1      | 0  LSB |
| -------- | ------ | ------ | ------ | ------ | ------ | ------ | ------ | ------ |
| MOSI (w) |        | x      | x      | x      | x      | x      | x      | x      |
| MISO (w) |        |        |        |        |        |        |        |        |

**4th byte:**

|          | 7  MSB | 6      | 5      | 4      | 3      | 2      | 1      | 0  LSB |
| -------- | ------ | ------ | ------ | ------ | ------ | ------ | ------ | ------ |
| MOSI (w) |        |        | y      | y      | y      | y      | y      | y      |
| MISO (w) |        |        |        |        |        |        |        |        |

**5th byte:**

|          | 7  MSB | 6      | 5      | 4      | 3      | 2      | 1      | 0  LSB |
| -------- | ------ | ------ | ------ | ------ | ------ | ------ | ------ | ------ |
| MOSI (w) | arca   | arca   | arca   | arca   | arca   | arca   | arca   | arca   |
| MISO (w) |        |        |        |        |        |        |        |        |

**6th byte:**

|          | 7  MSB | 6      | 5      | 4      | 3      | 2      | 1      | 0  LSB |
| -------- | ------ | ------ | ------ | ------ | ------ | ------ | ------ | ------ |
| MOSI (w) | arcb   | arcb   | arcb   | arcb   | arcb   | arcb   | arcb   | arcb   |
| MISO (w) |        |        |        |        |        |        |        |        |


**The 3rd, 4th, 5th and 6th bytes are repeated *count* times in unsorted order. (i.e. *3rd-4th-5th-6th-3rd-4th-5th-6th* for *count = 2*.)**

**The behaviour of reading is undefined.**

**The behaviour of *count>1* is undefined.**

*radius:* the radius (in pixels)

*count:* the number of focal points

*x:* the x (horizontal) coordinate of the focal point

*y:* the y (vertical) coordinate of the focal point

*arca:* the start of the visible arc

*arcb:* the end of the visible arc
- linear, clockwise
    - 0 is above and on the same x coordinate as the focal point
    - 192 is on the left side of and on the same y coordinate as the focal point
