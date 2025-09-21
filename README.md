# STM32 Flash Read/Write Example

## Description
This repository contains an example STM32 project demonstrating how to **write and read data in the internal flash memory**. The program uses UART (via `printf`) to display the data on a terminal (e.g., RealTerm). It shows how to safely erase a flash page, write 16-bit or 32-bit data, and read it back. 

The example is intended for educational purposes and demonstrates key concepts of flash memory usage on STM32 MCUs.

---

## Hardware Setup
- **MCU:** STM32 (any with internal flash, e.g., STM32F103C8T6)
- **UART Adapter:** NodeMCU (used for serial communication)
  - **RST ↔ GND** (to disable reset and use only UART)
  - **RX ↔ RX** (STM32)
  - **TX ↔ TX** (STM32)
- **Terminal Software:** RealTerm or any UART terminal (115200 baud, 8N1)

---

## Code Functionality

1. **Flash Write & Read Functions**
    ```c
    void flash_write_data(...);
    void flash_read_data(...);
    ```
    - `flash_write_data`: Unlocks flash, erases a page, writes 16-bit or 32-bit data, then locks the flash again.
    - `flash_read_data`: Reads back 16-bit or 32-bit data from the specified flash page and offset.

2. **Data Example**
    ```c
    uint32_t write_data[4] = {0x12345678, 0x02020202, 0x87654321, 0x04040404};
    uint32_t read_data[4];
    ```
    - `write_data` is written to flash.
    - `read_data` stores the values read back from flash.

3. **Main Loop**
    - Writes data to flash at startup.
    - Reads it back and prints to UART every second.

---

## Why Flash?
- Writing to flash allows **persistent storage** across power cycles.
- Unlike RAM variables, flash retains values even after MCU reset or power-off.
- This is useful for storing configuration data, calibration values, or logs.

---

## Flash Page Address Selection

```c
uint32_t page_addr = 0x08007C00;
```

The program memory usage from compilation is:
```c
Code = 4968 bytes
RO-data = 352 bytes
RW-data = 36 bytes
```
Total flash usage:
```c
4968 + 352 + 36 = 5356 bytes ≈ 0x14EC
```
- 4968 + 352 + 36 = 5356 bytes ≈ 0x14EC
- STM32 flash starts at 0x08000000. Therefore, the program ends roughly at 0x080014EC.
- To avoid overwriting program code, we select a flash page after the program data.
- 0x08007C00 is a safe page in the user flash area far enough from the code.
---
## Flash Functions

### `flash_write_data`

```c
void flash_write_data(
    uint32_t page_addr, 
    uint32_t offset_addr, 
    void* data, 
    uint16_t size, 
    DataTypeDef data_type_def
);
````

**Description:**
Writes data to a specific flash page at a given offset. Supports both 16-bit and 32-bit data types.

**Parameters:**

| Parameter       | Type          | Description                                             |
| --------------- | ------------- | ------------------------------------------------------- |
| `page_addr`     | `uint32_t`    | Base address of the flash page to write.                |
| `offset_addr`   | `uint32_t`    | Offset from the base page address where writing starts. |
| `data`          | `void*`       | Pointer to the data to write.                           |
| `size`          | `uint16_t`    | Number of elements to write.                            |
| `data_type_def` | `DataTypeDef` | Type of data: `DATA_TYPE_16` or `DATA_TYPE_32`.         |

**Steps inside the function:**

1. Unlocks flash memory for writing (`HAL_FLASH_Unlock()`).
2. Erases the specified flash page (`HAL_FLASHEx_Erase()`).
3. Programs each element in a loop:

   * 16-bit: `HAL_FLASH_Program(FLASH_TYPEPROGRAM_HALFWORD, ...)`
   * 32-bit: `HAL_FLASH_Program(FLASH_TYPEPROGRAM_WORD, ...)`
4. Locks the flash memory after writing (`HAL_FLASH_Lock()`).

**Note:** Always lock flash after write to prevent accidental overwrites.

---

### `flash_read_data`

```c
void flash_read_data(
    uint32_t page_addr, 
    uint32_t offset_addr, 
    void* data, 
    uint16_t size, 
    DataTypeDef data_type_def
);
```

**Description:**
Reads data from a specific flash page at a given offset. Supports both 16-bit and 32-bit data types.

**Parameters:**

| Parameter       | Type          | Description                                             |
| --------------- | ------------- | ------------------------------------------------------- |
| `page_addr`     | `uint32_t`    | Base address of the flash page to read.                 |
| `offset_addr`   | `uint32_t`    | Offset from the base page address where reading starts. |
| `data`          | `void*`       | Pointer to store the read data.                         |
| `size`          | `uint16_t`    | Number of elements to read.                             |
| `data_type_def` | `DataTypeDef` | Type of data: `DATA_TYPE_16` or `DATA_TYPE_32`.         |

**Steps inside the function:**

1. Calculates the actual memory address: `temp_addr = page_addr + offset_addr`.
2. Reads each element in a loop:

   * 16-bit: Reads half-word and increments address by 2.
   * 32-bit: Reads word and increments address by 4.
3. Stores the read values in the provided pointer `data`.

**Usage Example:**

```c
uint32_t write_data[4] = {0x12345678, 0x02020202, 0x87654321, 0x04040404};
uint32_t read_data[4];
flash_write_data(0x08007C00, 0, write_data, 4, DATA_TYPE_32);
flash_read_data(0x08007C00, 0, read_data, 4, DATA_TYPE_32);
```

* This writes 4 words to flash and reads them back to verify the operation.
