# Final Project — Manufacturing Test System

## Executive Summary

The implemented system is a manufacturing verification platform consisting of an STM32 UUT and a PC test application.

The PC sends a proprietary UDP command to the STM32. The STM32 validates the command, selects the requested peripheral test, performs the requested iterations, and returns a five-byte result containing the Test-ID and Success/Failure status.

The implemented peripheral tests are TIMER, UART, SPI, I2C and ADC. Ethernet/LwIP provides the communication path.

## Requirements Coverage

| Requirement | Status | Verification |
|---|---|---|
| Ethernet MAC/PHY | PASS | Ethernet/UDP communication |
| UDP/IP communication | PASS | Commands and responses |
| Proprietary command format | PASS | Test-ID/peripheral/iterations/pattern |
| Result format | PASS | 5-byte response |
| TIMER test | PASS | TIM1 update-event counting |
| UART test | PASS | UART4 <-> UART5 DMA loopback |
| UART DMA | PASS | DMA TX/RX |
| UART CRC for >100 bytes | PASS | 128-byte test, CRC32 match |
| SPI test | PASS | SPI4 master / SPI1 slave |
| SPI DMA | PASS | DMA transfer and comparison |
| I2C test | PASS | I2C1 master / I2C4 slave |
| I2C DMA/interrupts | PASS | I2C1 DMA, I2C4 IRQ |
| ADC test | PASS | ADC1 12-bit, PA0 |
| PC CLI | PASS | C++17 Linux client |
| Persistent records | PASS | SQLite |
| Test duration | PASS | Measured send-to-response |
| Success/Failure | PASS | 0x01 / 0xFF |
| Edge-case handling | PASS | Invalid commands tested |
| Doxygen | REVIEW | Public headers should be checked |
| Board documentation | REQUIRED | F746 assignment vs F756 implementation |

## UART CRC Verification

A 128-byte UART pattern was used.

Since:

```text
128 > 100
```

the UART test selected the CRC32 verification path.

Observed result:

```text
CRC32: PATTERN=24650D57
       UART5_RX=24650D57
       UART4_RX=24650D57

UART CRC32 CHECK: PASS
```

The test completed all three iterations successfully.

## Edge Cases

The implementation was tested with:

1. `iterations = 0`
2. unknown peripheral value
3. invalid pattern length
4. oversized UDP command

The server returned `0xFF` and did not execute the invalid peripheral test.

## PC-side persistence

The PC application records:

- Test-ID
- date/time sent
- test duration
- result
- peripheral
- iterations
- pattern

The C++ client stores these records in SQLite and supports:

```text
all
run
history
get
```

## Hardware Target Note

The assignment specifies STM32F746ZG. The physically verified implementation uses NUCLEO-F756ZG / STM32F756ZGTx. The implementation was adapted to the actual target and its peripheral/pin configuration.

## Conclusion

The functional STM32 manufacturing-test chain has been integrated and exercised over Ethernet/UDP for all five required peripheral tests. UART has additionally been verified with the required large-data CRC32 path. A C++17 Linux PC client is provided for the formal PC-side implementation, while the Python client can remain as an engineering/debug utility.

Final pre-submission work should focus on reviewing Doxygen coverage, placing the verified source files into the final repository structure, and performing one clean end-to-end demonstration from the Linux client.
