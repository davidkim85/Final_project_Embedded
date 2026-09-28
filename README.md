# STM32 Embedded Systems Manufacturing Test Project

## 1. Project purpose

The project implements a manufacturing verification system in which a PC sends test commands to an STM32 UUT over Ethernet/UDP. The STM32 parses the command, selects the requested peripheral test, executes the requested number of iterations, and returns a Success/Failure result.

The assignment specifies C/C++ on Linux for the PC test program and requires persistent test records.

## 2. Hardware note

The assignment document specifies STM32F746ZG. The implemented and physically verified target is:

- NUCLEO-F756ZG
- STM32F756ZGTx

The peripheral configuration and pin mapping were adapted to the actual F756 target.

## 3. System architecture

```text
                    PC / Linux
                       |
                       | UDP / IPv4
                       |
                       v
              +------------------+
              | Ethernet / LwIP  |
              +--------+---------+
                       |
                       v
                 udp_server.c
                       |
                       v
                  protocol.c
                       |
                       v
                 test_engine.c
                       |
       +---------------+---------------+
       |       |       |       |       |
       v       v       v       v       v
     TIMER   UART    SPI     I2C      ADC
```

## 4. UDP protocol

### Command

```text
Byte 0..3 : Test-ID, little-endian
Byte 4    : Peripheral
Byte 5    : Iterations
Byte 6    : Pattern length
Byte 7..  : Pattern
```

Peripheral values:

```text
0x01 TIMER
0x02 UART
0x04 SPI
0x08 I2C
0x10 ADC
```

Maximum command size:

```text
7 + 255 = 262 bytes
```

### Result

```text
Byte 0..3 : Test-ID, little-endian
Byte 4    : Result
```

Result:

```text
0x01 SUCCESS
0xFF FAILURE
```

## 5. STM32 peripheral tests

### TIMER

TIM1 is used as the timer under test.

Configuration:

```text
Timer clock  = 72 MHz
Prescaler    = 71
Counter      = 1 MHz
Period       = 999
Update event = 1 ms
```

The test counts TIM1 update callbacks for the requested number of events.

### UART

UART4 and UART5 are used as a loopback pair.

```text
UART4 TX PC10 -> UART5 RX PD2
UART5 TX PC12 -> UART4 RX PC11
GND            -> GND
```

Configuration:

```text
115200 baud
8 data bits
1 stop bit
No parity
DMA
```

Data path:

```text
UART4 -> UART5 -> UART4
```

For pattern length <= 100 bytes, the received data is checked byte-by-byte.

For pattern length > 100 bytes, CRC32 is calculated for:

```text
Original pattern
UART5 received data
UART4 returned data
```

All three CRC32 values must match.

### SPI

SPI4 is master and SPI1 is slave.

```text
SPI4 PE2 SCK  -> SPI1 PA5 SCK
SPI4 PE6 MOSI -> SPI1 PB5 MOSI
SPI4 PE5 MISO <- SPI1 PA6 MISO
SPI4 PE4 NSS  -> SPI1 PA4 NSS
GND           -> GND
```

Both sides use 8-bit data and DMA.

### I2C

I2C1 is master and I2C4 is slave.

```text
I2C1 PB8 SCL <-> I2C4 PF14 SCL
I2C1 PB9 SDA <-> I2C4 PF15 SDA
GND            <-> GND
```

I2C1 uses DMA. I2C4 uses interrupt-driven operation.

### ADC

ADC1 channel 0 is connected to PA0.

Configuration:

```text
12-bit resolution
ADC1_IN0
```

For the verified setup PA0 is connected to ground. The expected conversion is therefore approximately zero. A tolerance of 64 ADC counts is used to account for small measured noise.

## 6. Ethernet / LwIP

```text
IP      : 192.168.1.198
Netmask : 255.255.255.0
Gateway : 192.168.1.1
UDP     : 5000
```

DHCP is disabled.

The UDP server starts after peripheral initialization and processes packets from the main loop.

## 7. PC client

The formal PC implementation is C++17 for Linux.

Commands:

```bash
./manufacturing_test all

./manufacturing_test run \
  --id 9001 \
  --peripheral uart \
  --iterations 3 \
  --pattern "11 22 33 44 55"

./manufacturing_test history

./manufacturing_test get --id 9001
```

SQLite database:

```text
tests_cpp.db
```

Stored information:

- Test-ID
- date/time sent
- test duration in seconds
- result
- peripheral
- iterations
- pattern

The existing Python client may be retained as a debug/engineering test utility.

## 8. Verified test results

The complete UDP integration sequence was successfully executed for:

```text
TIMER
UART
SPI
I2C
ADC
```

Three iterations were used.

UART was additionally verified with a 128-byte pattern. Because 128 > 100, the CRC32 path was selected and the console showed equal CRC32 values for the original pattern, UART5 receive buffer and UART4 receive buffer.

## 9. Edge cases verified

The following invalid inputs were tested:

- iterations = 0
- unknown peripheral value
- invalid pattern length for TIMER
- oversized UDP command

The STM32 returned the Failure result (`0xFF`) and did not start an invalid peripheral test.

## 10. Documentation / Doxygen

Public functions should use Doxygen comments in the following form:

```c
/**
 * @brief Runs the UART manufacturing test.
 *
 * @param pattern Test data pattern.
 * @param length Pattern length in bytes.
 * @param iterations Number of iterations.
 *
 * @return true if all iterations pass.
 * @return false if any iteration fails.
 */
bool UART_Test_Run(const uint8_t *pattern,
                   uint8_t length,
                   uint8_t iterations);
```

## 11. Final project structure

```text
Final_Project/
├── STM32/
│   ├── Core/
│   ├── protocol/
│   ├── udp/
│   ├── tests/
│   │   ├── test_engine.c/.h
│   │   ├── timer_test.c/.h
│   │   ├── uart_test.c/.h
│   │   ├── spi_test.c/.h
│   │   ├── i2c_test.c/.h
│   │   └── adc_test.c/.h
│   └── ...
├── PC/
│   ├── include/
│   ├── src/
│   ├── Makefile
│   └── README.md
└── tools/
    └── test_client.py
```

## 12. Submission note

Before submission, ensure the final STM32 CubeIDE project contains the verified source files and that the C++ Linux client is built and demonstrated on Linux.
