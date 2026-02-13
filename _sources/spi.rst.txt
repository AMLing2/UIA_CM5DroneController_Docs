=======================================================
1. Internal SPI interfaces
=======================================================
Sensors
-----------------
The barometer (BMP384) and IMU (BMI088) are both connected to SPI port 1, the barometer uses Chip Select (CS) SPI1_CE2_N Pin 29/GPIO 16. The barometer's interrupt pin is placed on Pin 31/GPIO 12.
The IMU uses CS SPI1_CE1_N Pin 50/GPIO 17 for the accelerometer, and CS SPI1_CE0_N Pin 49/GPIO 18 for the gyroscope. INT1 for the acelerometer is placed on Pin 54/GPIO 4, and INT3 for hthe gyroscope is placed on Pin 55/GPIO 14.
See the respective datasheets for SPI communication with the sensors.

-----------------------
MCU Motor Controller
-----------------------
The ATMEGA32u4 is placed on SPI port 0, using CS SPI0_CE0_N Pin 39/GPIO 8, SPI Mode 0. The MCU can be programmed using the JTAG or ISP header, or by using a USB bootloader.
Each SPI message should contain a header byte which includes the command in the first 3 bits, and a set length of the payload in the last 5 bits, differing values are rejected, the header values are defined in Table 1.1. The header and payload should be followed by a 2 byte CRC-16 code, which uses CRC-16-CCITT polynomials.

.. rubric:: Table 1.1 Header command values.
.. list-table::
   :header-rows: 1
   :widths: 25 10 25 40

   * - Description
     - Command
     - Payload length
     - Hex value
   * - 9bit PWM Motor Data
     - 1
     - 5
     - 0x25
   * - Battery reading reply
     - 2
     - 3
     - 0x43
   * - Read battery status
     - 3
     - 0
     - 0x60

.. rubric:: Table 1.2 header byte explanation
.. list-table::
   :header-rows: 1
   :widths: 4 4 4 4 4 4 4 4

   * - 7
     - 6
     - 5
     - 4
     - 3
     - 2
     - 1
     - 0
   * - CMD2
     - CMD1
     - CMD0
     - Len4
     - Len3
     - Len2
     - Len1
     - Len0


After requesting a battery reading, the data will be sent an unspecified amount of time later. The SPI port can be continously monitored where 0x00 will be sent if no data is availible, otherwise the battery data will be sent synchronously during other SPI communications. The SPI port can still be used for sending motor signals during this step.
The specific payload bytes and full examples are given in Tables 1.2-1.4.

.. rubric:: Table 1.2 9Bit PWM Motor Data payload.
.. list-table::
   :header-rows: 1
   :widths: 6 30 30

   * - Byte
     - Description
     - Example
   * - 1
     - Header
     - 0x25
   * - 2
     - bits 0-3 contains MSB bit of motors 1-4, respectively, first four MSB bits are unused
     - 0b0000 1010, Motors 1 and 3 on <256 (0x0nn), motors 2 and 4 on >256 (0x1nn)
   * - 3
     - 8 bit LSB Motor 1
     - 0xFF (0x0FF including above - half speed)
   * - 4
     - 8 bit LSB Motor 2
     - 0xFF (0x1FF - max speed)
   * - 5
     - 8 bit LSB Motor 3 
     - 0x00 (0x000 - motor off)
   * - 6
     - 8 bit LSB Motor 4
     - 0x55 (0x155 ~ 75% of max)
   * - 7
     - MSB of CRC-16
     - 0xB0 (From above sequence)
   * - 8
     - LSB of CRC-16
     - 0x27


.. rubric:: Table 1.3 Battery request.
.. list-table::
   :header-rows: 1
   :widths: 6 10 30

   * - Byte
     - Description
     - Example
   * - 1
     - Header
     - 0x60
   * - \-
     - <No payload>
     - \-
   * - 2
     - MSB of CRC-16
     - 0x8D
   * - 3
     - LSB of CRC-16
     - 0x56

.. rubric:: Table 1.6 test.
.. list-table::
   :header-rows: 0
   :widths: 4 4 4 4 4 4 4 4

   * - \-
     - \-
     - \-
     - \-
     - V9
     - V8
     - C9
     - C8


.. rubric:: Table 1.4 Battery recieve sequence.
.. list-table::
   :header-rows: 1
   :widths: 6 30 10

   * - Byte
     - Description
     - Example
   * - 1
     - Header
     - 0x43
   * - 2
     - LSB bits 0:1 are the MSB of the current, bits 2:3 are MSB of voltage
     - todo
   * - 3
     - LSB of 10 bit voltage reading
     - todo
   * - 4
     - LSB of 10 bit current reading
     - todo
   * - 5
     - MSB of CRC-16
     - todo
   * - 6
     - LSB of CRC-16
     - todo
