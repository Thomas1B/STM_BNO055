# STM BNO055 Sensor 

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
