
# AD5761_Micropython - AD5761 DAC Control on MicroPython

This project provides a full implementation for controlling **AD5761** Digital-to-Analog Converter (DAC) using **MicroPython**. All testing is done on a **Raspberry Pi Pico** with RP2040. You might want to add timing interval between sync pulses and read/write operations if you encounter random bit returns.

## Features

- **Control Register Operations**: Set/Read the control register with the desired configuration.
- **DAC register Operations**: Write, update and read the DAC register using percentages of the full scale.
- **Input Register Operations**: Write, update and read to the input register without immediately updating the DAC, then update the DAC register from the input register.
- **Daisy-Chain**: Enable or disable daisy-chain functionality. Readback is disabled when daisy-chain is disabled.
- **Reset**: Perform a full reset or data reset to a power-up scale defined in the control register.

## Requirements

- **AD5761 DAC** on a breakout board
- **MicroPython** on the Microcontroller
- **Raspberry Pi Pico** (or another supported microcontroller)
- SPI interface setup on the microcontroller

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

### Installing MicroPython

1. Use Thonny IDE(<https://thonny.org/>) to flash the MicroPython into the Pico.

### Uploading the Code

1. Clone the repository:

   ```bash
   git clone https://github.com/intel00000/ad5761_micropython.git
   ```

2. Upload the `AD5761.py` to the MCU.

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

### Writing to the DAC Register

Write a percentage value to the DAC register and update the output:

```python
# Write and update DAC with 50% of full scale
ad5761.write_update_dac_register(50)
```

### Writing to the Input Register

Write to the input register without updating the DAC (this depend on the LDAC pin value, check the spec sheet for more information):

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

To enable detailed debugging output, pass `debug=True` when initializing the `AD5761Controller`. This will print SPI commands, register values, and alert pin status during execution.

## Example

```python
# Example Usage
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
