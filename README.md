# STM BNO055 Sensor

using: https://github.com/ivyknob/bno055_stm32/tree/master

datasheet: [BNO055](https://www.bosch-sensortec.com/media/boschsensortec/downloads/datasheets/bst-bno055-ds000.pdf#%5B%7B%22num%22%3A33%2C%22gen%22%3A0%7D%2C%7B%22name%22%3A%22XYZ%22%7D%2C68%2C715%2C0%5D)

## BNO0555 Features

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
In Stm32CubeMX, pick what I2C pins you want to use (leave everything default)

### System Error Codes
This are read by using `printf("Error: %d \r\n", bno055_getSystemError());`

- `0x00` - NO ERROR
- `0x01` - PERIPHERAL INITIALIZATION ERROR
- `0x02` - SYSTEM INITIALIZATION ERROR
- `0x03` - SELF TEST FAILED
- `0x04` - MAP VAL OUT OF RANGE
- `0x05` - MAP ADDR OUT OF RANGE
- `0x06` - REG MAP WRITE ERROR
- `0x07` - LOW PWR MODE NOT AVAILABLE FOR SELECTED OPR MODE
- `0x08` - ACCEL PWR MODE NOT AVAILABLE
- `0x09` - FUSION ALGO CONF ERROR
- `0x0A` - SENSOR CONF ERROR
