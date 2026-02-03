=======================================================
1. Internal SPI interfaces
=======================================================
Sensors
--------
The barometer (BMP384) and IMU (BMI088) are both connected to SPI port 1, the barometer uses Chip Select (CS) Pin 29/GPIO 16. The barometer's interrupt pin is placed on Pin 31/GPIO 12.
The IMU uses CS Pin 50/GPIO 17 for the accelerometer, and CS Pin 49/GPIO 18 for the gyroscope. INT1 for the acelerometer is placed on Pin 54/GPIO 4, and INT3 for hte gyroscope is placed on Pin 55/GPIO 14.
See the respective datasheets for SPI communication with the sensors.

--------
MCU Motor Controller
--------
The ATMEGA32u4 is placed on SPI port 0, using CS Pin 39/GPIO 8, SPI Mode 0.
Each message should contain a header byte including the command and a set length of the payload, differing values are rejected, the header values are defined in Table 1.1. The header and payload should be followed by a 2 byte CRC-16 code, which uses CRC-16-CCITT polynomials. After requesting a battery reading, the data will be ready to be sent an unspecified amount of time later. The SPI port can be continously monitored where 0x00 will be sent if no data is availible, otherwise the battery data will be sent synchronously during other SPI communications. The SPI port can still be used for sending motor signals during this step.
The specific payload bytes and full examples are given in Tables 1.2-1.4.

| Description | Command | Payload length | Hex value |
|---|---:|---:|---|
| 9bit PWM Motor Data | 1 | 5 | `0x03 0x00 0x10 0x00 0x00`  — header `0x03`, address `0x0010`, read 2 bytes (zeros are clocked out to receive data) |
| Read battery status | 2 | 0 | `0x02 0x00 0x10 0xDE 0xAD` — header `0x02`, write `0xDEAD` to address `0x0010` |
| Battery reading reply (sent from MCU) | 3 | 3 | `0x02 0x00 0x10 0xDE 0xAD` — header `0x02`, write `0xDEAD` to address `0x0010` |
  Table 1.1 Header command values.

| Byte | Description | Example |
|---|---:|---|
| 1 | Header | |
| 2 | First four MSB bits are unused, bits 0-3 contains the MSB bit of motors 1-4 respectively  | 0b0000 1010, Motors 1 and 3 on <256 (0x0nn), motors 2 and 4 on >256 (0x1nn) |
| 3 | 8 bit LSB Motor 1  | 0xFF (0x0FF including above - half speed) |
| 4 | 8 bit LSB Motor 2  | 0xFF (0x1FF - max speed) |
| 5 | 8 bit LSB Motor 3  | 0x00 (0x000 - motor off) |
| 6 | 8 bit LSB Motor 4  | 0x55 (0x155 ~ 75% of max) |
| 7 | MSB of CRC-16  | - |
| 8 | LSB of CRC-16  | - |
  Table 1.2 9Bit PWM Motor Data payload.


| Byte | Description | Example |
|---|---:|---|
| 1 | Header | |
| - | <No payload> | - |
| 2 | MSB of CRC-16  | - |
| 3 | LSB of CRC-16  | - |
  Table 1.3 Battery request.

| Byte | Description | Example |
|---|---:|---|
| 1 | Header | |
| - | <No payload> | - |
| 2 | MSB of CRC-16  | - |
| 3 | LSB of CRC-16  | - |
  Table 1.4 Battery recieve sequence.
