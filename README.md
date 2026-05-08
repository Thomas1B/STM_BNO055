# STM BNO055 Sensor

Built from: https://github.com/ivyknob/bno055_stm32/tree/master

Sensor datasheet: [BNO055](https://www.bosch-sensortec.com/media/boschsensortec/downloads/datasheets/bst-bno055-ds000.pdf#%5B%7B%22num%22%3A33%2C%22gen%22%3A0%7D%2C%7B%22name%22%3A%22XYZ%22%7D%2C68%2C715%2C0%5D)

## BNO055 Features

- Sensor data:
  - Acceleration $\rightarrow m/s^2$
  - Magnetometer $\rightarrow \mu T$
  - Gyroscope $\rightarrow rad/s$
  - Euler Angles $\rightarrow (^{\circ} \space \space degrees)$
  - Quaternion $\rightarrow$ No Units
  - Linear Acceleration $\rightarrow m/s^2$
  - Gravity $\rightarrow m/s^2$
- Calibration Support:
  - get calibration state
  - Read/write calibration data
- System Diagnostic:
  - Self-test results
  - System status & error codes

## Setup
- Copy `bno055.c`, `bno055.h`, and `bno055_stm32.h` to your project.
- In Stm32CubeMX, pick what I2C pins you want to use (BNO0555 is limited to 100Hz, so leave everything default, Standard Mode works fine).
- Add `#include "bno055_stm32.h"` to "USER CODE BEGIN Includes".
- Set I2C address `#define BNO055_I2C_ADDR` in bno055.h:
```C
#define BNO055_I2C_ADDR    BNO055_I2C_ADDR_LO    // For 0x28
#define BNO055_I2C_ADDR    BNO055_I2C_ADDR_HI    // For 0x29
```

- Add the following to "USER CODE BEGIN Includes"
```C
#include "bno055_stm32.h"
#include <stdio.h>
```

- Add the following to "USER CODE BEGIN 2"
```C
bno055_assignI2C(&hi2c1); // Assign the I2C handle to the BNO055 library
bno055_setup(); // Initialize the BNO055 sensor
bno055_setOperationMode(BNO055_OPERATION_MODE_NDOF); // Set the operation mode to NDOF (fusion mode)

// Optional test print
// Scan the I2C bus for devices and print their addresses
for (uint8_t addr = 1; addr < 128; addr++) {
  if (HAL_I2C_IsDeviceReady(&hi2c1, addr << 1, 1, 10) == HAL_OK) {
    printf("Device found at address:0x%02X\r\n", addr);
  }
}

printf("Status: %d \r\n", bno055_getSystemStatus());
if (bno055_getSystemStatus() != 0) {
  printf("System Error: %d \r\n", bno055_getSystemError());
}
HAL_Delay(2000);
```
- Add to USER CODE BEGIN 3
```C
// Add this to "User Code Begin 3"
bno055_vector_t v = bno055_getVectorEuler();
printf("Heading: %.2f Roll: %.2f Pitch: %.2f\r\n", v.x, v.y, v.z);
HAL_Delay(1000);
```

Now test the code...

<hr>

## Operation Modes


| Mode Name      | Value (hex) | What it does                                           |
|----------------|------------|--------------------------------------------------------|
| `CONFIGMODE`   | `0x00`     | Configuration mode (required before changing settings) |
| `ACCONLY`      | `0x01`     | Accelerometer only                                     |
| `MAGONLY`      | `0x02`     | Magnetometer only                                      |
| `GYRONLY`      | `0x03`     | Gyroscope only                                         |
| `ACCMAG`       | `0x04`     | Accelerometer + Magnetometer                           |
| `ACCGYRO`      | `0x05`     | Accelerometer + Gyroscope                              |
| `MAGGYRO`      | `0x06`     | Magnetometer + Gyroscope                               |
| `AMG`          | `0x07`     | Accel + Mag + Gyro (no fusion)                         |
| `IMUPLUS`      | `0x08`     | Accel + Gyro (fusion, no mag)                          |
| `COMPASS`      | `0x09`     | Accel + Mag (fusion for heading)                       |
| `M4G`          | `0x0A`     | Mag + Accel (gyro-free fusion)                         |
| `NDOF_FMC_OFF` | `0x0B`     | Full fusion (no fast mag calibration)                  |
| `NDOF`         | `0x0C`     | Full 9-DOF sensor fusion                               |

This is set by 
```C
bno055_setOperationMode(CONFIGMODE);
bno055_setOperationMode(mode); // new mode
```


## System Status Codes

| Value | Hex Value | Description                              |
| ----- | --------- | ---------------------------------------- |
| 0     | `0x00`    | Idle                                     |
| 1     | `0x01`    | System error                             |
| 2     | `0x02`    | Initializing peripherals                 |
| 3     | `0x03`    | System initialization                    |
| 4     | `0x04`    | Executing self-test                      |
| 5     | `0x05`    | Sensor fusion algorithm running          |
| 6     | `0x06`    | System running without fusion algorithms |

This is read by 
```C
uint8_t bno055_getSystemStatus();
```



## System Error Codes


| Code (hex) | Description                                      |
|------------|--------------------------------------------------|
| `0x00`     | No error                                         |
| `0x01`     | Peripheral initialization error                  |
| `0x02`     | System initialization error                      |
| `0x03`     | Self test failed                                 |
| `0x04`     | Map value out of range                           |
| `0x05`     | Map address out of range                         |
| `0x06`     | Register map write error                         |
| `0x07`     | Low power mode not available for selected mode   |
| `0x08`     | Accelerometer power mode not available           |
| `0x09`     | Fusion algorithm configuration error             |
| `0x0A`     | Sensor configuration error                       |

This is read by using 
```C
printf("Error: %d \r\n", bno055_getSystemError());`
```

## Remapping Coordinate Axes
Axis remapping changes which physical sensor axis is used as
the logical X, Y, and Z axes.

Axis sign mapping allows individual axes to be inverted.
 
The BNO055 should be placed into CONFIGMODE before calling
this function.

Axis_Map_Config:
| Binary Value | Decimal Value | Axis Representation |
| ------------ | ------------- | ------------------- |
| `00`         | `0`           | X - Axis            |
| `01`         | `1`           | Y - Axis            |
| `10`         | `2`           | Z - Axis            |
| `11`         | `3`           | Invalid             |

Axis_Sign_Map:
| Value | Sign     |
| ----- | -------- |
| `0`   | Positive |
| `1`   | Negative |


 Example:
 ```C
  bno055_axis_map_t axisMap = {
      .x = 1,
      .y = 0,
      .z = 2,
      .x_sign = 0,
      .y_sign = 1,
      .z_sign = 0
  };
 
  bno055_setOperationMode(CONFIGMODE);
  bno055_setAxisMap(axisMap);
  bno055_setOperationMode(NDOF); // or what mode yyou prefer.
 ```
