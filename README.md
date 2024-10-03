
# AD5761_Micropython - AD5761 DAC Control on MicroPython

This project provides a full implementation for controlling the **AD5761** Digital-to-Analog Converter on **MicroPython**. All testing is done on a **Raspberry Pi Pico** with RP2040. You might want to add timing interval between sync pulses and read/write operations if you encounter random bit in return values.
Datasheet Reference: [AD5761 Datasheet](./ad5761_5721.pdf)

## Features

- **Control Register Operations**: Set/Read the control register with the desired configuration.
- **DAC register Operations**: Write, update and read the DAC register using percentages of the full scale.
- **Input Register Operations**: Write and update to the input register without immediately updating the DAC, then update the DAC register from the input register.
- **Daisy-Chain**: Enable or disable daisy-chain functionality. Readback is disabled when daisy-chain is disabled.
- **Reset**: Perform a full reset or data reset to a power-up scale defined in the control register.

## Requirements

- **AD5761 DAC** on a breakout board
- **MicroPython** on the Microcontroller
- **Raspberry Pi Pico** (or another supported microcontroller)

## Getting Started

### Hardware Setup

Connect the Raspberry Pi Pico to the AD5761 DAC using the following GPIO pins:

| Pico Pin  | AD5761 Pin |
|-----------|------------|
| `sclk_pin` (e.g., GPIO 6) | SPI Clock (SCLK) |
| `sdi_pin` (e.g., GPIO 7)  | SPI Data Input (SDI) |
| `sdo_pin` (e.g., GPIO 4)  | SPI Data Output (SDO) |
| `sync_pin` (e.g., GPIO 5) | SPI Chip Select (SYNC) |
| `alert_pin` (e.g., GPIO 12) | Alert (optional, high when DAC is ready, low when DAC is not ready or in fault condition) |

## Usage

Below is an overview of its key methods:

### Initialization

```python
from AD5761 import AD5761

# Initialize Controller
ad5761 = AD5761(
    sclk_pin=6,       # SPI Clock Pin
    sdi_pin=7,        # SPI Data Input Pin
    sdo_pin=4,        # SPI Data Output Pin
    sync_pin=5,       # SPI Chip Select (SYNC) Pin
    alert_pin=12,     # Alert Pin (optional, needed for fault detection)
    debug=True        # Enable debugging
)
```

### Full Reset

Perform a full software reset:

```python
ad5761.reset()
```

### Writing to Control Register

Write to the control register with the desired configuration:

```python
# Write to control register with full scale, overrange disabled, straight binary, thermal shutdown enabled, zero scale power-up, 0V to 10V range
ad5761.write_control_register(
    cv=ad5761.CV_FULL_SCALE,
    ovr=ad5761.OVR_DISABLED,
    b2c=ad5761.B2C_BINARY,
    ets=ad5761.ETS_POWER_DOWN,
    pv=ad5761.PV_ZERO_SCALE,
    ra=ad5761.RA_10V_UNIPOLAR,
)
```

options are available in the [Control Register Options](#control-register-options) section.

### Writing to DAC Register

Write a percentage value to the DAC register and update the output:

```python
# Write and update DAC with 50% of full scale
ad5761.write_update_dac_register(50)
```

### Writing to Input Register

Write to the input register without updating the DAC (this also depend on the LDAC pin value, check the spec sheet for more information):

```python
# Write 25% to the input register
ad5761.write_input_register(25)
```

### Update DAC from Input Register

Update the DAC from the input register:

```python
ad5761.update_dac_from_input_register()
```

### Reading Registers

Read and print the DAC register value:

```python
# Read DAC register and print its value
ad5761.read_dac_register()
```

Read and print the control register with detailed interpretation:

```python
ad5761.read_control_register()
```

### Daisy-Chain Control

Enable or disable daisy-chaining, readback is disabled when daisy-chain is disabled:

```python
# Enable daisy-chain functionality
ad5761.set_daisy_chain(enabled=True)

# Disable daisy-chain functionality
ad5761.set_daisy_chain(enabled=False)
```

### Software Data Reset

Perform a software data reset, resetting the DAC output to the power-up voltage set in the control register:

```python
ad5761.software_data_reset()
```

### Debug Mode

Pass `debug=True` during initialization for detailed debugging information:

## Control Register Options

Below are configuration options for write_control_register function.

### CV (Clear Voltage Selection)

- `CV_ZERO_SCALE (0x0)` - Clear to zero scale.
- `CV_MIDSCALE (0x1)` - Clear to midscale.
- `CV_FULL_SCALE (0x2)` - Clear to full scale.

### OVR (Overrange Enable)

- `OVR_DISABLED (0x0)` - Overrange disabled.
- `OVR_ENABLED (0x1)` - Overrange enabled (up to 5% overrange).

### B2C (Bipolar Range Coding)

- `B2C_BINARY (0x0)` - DAC input for bipolar output range is straight binary coded.
- `B2C_TWOS_COMPLEMENT (0x1)` - DAC input for bipolar output range is two's complement coded.

### ETS (Thermal Shutdown Enable)

- `ETS_NOT_POWER_DOWN (0x0)` - The internal digital supply does not power down if die temperature exceeds 150°C.
- `ETS_POWER_DOWN (0x1)` - The internal digital supply powers down if die temperature exceeds 150°C.

### PV (Power-Up Voltage)

- `PV_ZERO_SCALE (0x0)` - DAC powers up at zero scale.
- `PV_MIDSCALE (0x1)` - DAC powers up at midscale.
- `PV_FULL_SCALE (0x2)` - DAC powers up at full scale.

### RA (Output Range Selection)

- `RA_10V_BIPOLAR (0x0)` - Output range: −10V to +10V.
- `RA_10V_UNIPOLAR (0x1)` - Output range: 0V to +10V.
- `RA_5V_BIPOLAR (0x2)` - Output range: −5V to +5V.
- `RA_5V_UNIPOLAR (0x3)` - Output range: 0V to +5V.
- `RA_7_5V (0x4)` - Output range: −2.5V to +7.5V.
- `RA_3V (0x5)` - Output range: −3V to +3V.
- `RA_16V_UNIPOLAR (0x6)` - Output range: 0V to +16V.
- `RA_20V_UNIPOLAR (0x7)` - Output range: 0V to +20V.

## testing example

```python
ad5761 = AD5761Controller(sclk_pin=6, sdi_pin=7, sdo_pin=4, sync_pin=5, alert_pin=12, debug=True)
# Perform a reset
ad5761.reset()
time.sleep(1)
# Configure DAC control register
ad5761.write_control_register(cv=ad5761.CV_FULL_SCALE, ra=ad5761.RA_10V_UNIPOLAR)
time.sleep(1)
# Write 50% value to DAC register
ad5761.write_update_dac_register(50)
time.sleep(1)
# Write 25% to input register without updating DAC
ad5761.write_input_register(25)
time.sleep(1)
# Update DAC from input register
ad5761.update_dac_from_input_register()
time.sleep(1)
# Perform a software data reset
ad5761.software_data_reset()
```

## License

This project is licensed under the MIT License - see the [LICENSE](./LICENSE) file for details.
