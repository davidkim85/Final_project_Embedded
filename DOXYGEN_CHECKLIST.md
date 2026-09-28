# Doxygen checklist for the STM32 project

Use Doxygen comments for every public function in `.h` files and for important public data structures.

## Function template

```c
/**
 * @brief Short description.
 *
 * Detailed explanation of what the function does.
 *
 * @param name Description of the parameter.
 *
 * @return Description of the return value.
 */
```

## Example: protocol

```c
/**
 * @brief Parses one UDP manufacturing-test command.
 *
 * Validates packet size, peripheral value, iteration count and
 * pattern length before copying the command fields.
 *
 * @param data UDP payload.
 * @param length UDP payload length.
 * @param command Output command structure.
 *
 * @return true when the command is valid.
 * @return false when validation fails.
 */
bool Protocol_ParseCommand(const uint8_t *data,
                           uint16_t length,
                           Protocol_Command_t *command);
```

## Example: test engine

```c
/**
 * @brief Executes the peripheral test selected by the command.
 *
 * @param command Parsed manufacturing-test command.
 *
 * @return true if the selected test passes.
 * @return false if the command is invalid or the test fails.
 */
bool Test_Engine_Run(const Protocol_Command_t *command);
```

## Example: CRC32

```c
/**
 * @brief Calculates CRC32 over a byte buffer.
 *
 * @param data Input data.
 * @param length Number of bytes.
 *
 * @return Calculated CRC32 value.
 */
uint32_t CRC32_Calculate(const uint8_t *data,
                         uint32_t length);
```

## Files to review

```text
protocol.h
udp_server.h
test_engine.h
timer_test.h
uart_test.h
spi_test.h
i2c_test.h
adc_test.h
crc32.h
```

The generated CubeMX files should retain their generated structure and USER CODE sections.
