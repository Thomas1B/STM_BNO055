# STM BNO055 Sensor

Built from: https://github.com/ivyknob/bno055_stm32/tree/master

Sensor datasheet: [BNO055](https://www.bosch-sensortec.com/media/boschsensortec/downloads/datasheets/bst-bno055-ds000.pdf#%5B%7B%22num%22%3A33%2C%22gen%22%3A0%7D%2C%7B%22name%22%3A%22XYZ%22%7D%2C68%2C715%2C0%5D)

## BNO055 features

- Sensor data:
  - Acceleration 
  - Magnetometer 
  - Gyroscope 
  - Euler Angles
  - Quaternion 
  - Linear Acceleration
  - Gravity 
- Calibration Support:
  - get calibration state
  - Read/write calibration data
- System Diagnostic:
  - Self-test results
  - System status & error codes

## Setup
- Copy `bno055.c`, `bno055.h`, and `bno055_stm32.h` to your project.
- In STM32CubeMX, pick what I2C pins you want to use (BNO0555 is limited to 100Hz, so leave everything default, Standard Mode works fine).
- Add `#include "bno055_stm32.h"` to "USER CODE BEGIN Includes".
- Set I2C address `#define BNO055_I2C_ADDR` in bno055.h:
```C
#define BNO055_I2C_ADDR_HI 0x29
#define BNO055_I2C_ADDR_LO 0x28 // this is default addr
#define BNO055_I2C_ADDR    BNO055_I2C_ADDR_LO
```

- Add to USER CODE BEGIN Includes
```C
#include "bno055_stm32.h"
#include <stdio.h> // this is for using printf
```

- Add to USER CODE BEGIN 2
```C
// Optional test print
// Scan the I2C bus for devices and print their addresses
for (uint8_t addr = 1; addr < 128; addr++) {
  if (HAL_I2C_IsDeviceReady(&hi2c1, addr << 1, 1, 10) == HAL_OK) {
    printf("Device found at address:0x%02X\r\n", addr);
  }
}

bno055_assignI2C(&hi2c1); // Assign the I2C handle to the BNO055 library
bno055_setup(); // Initialize the BNO055 sensor
bno055_setOperationMode(BNO055_OPERATION_MODE_NDOF); // Set the operation mode to NDOF (fusion mode)
// bno055_setCalibrationData(savedCalData); // read calibration set up in READNE to use this.

HAL_Delay(2000);
```
- Add to USER CODE BEGIN 3
```C
// Add this to "User Code Begin 3"
bno055_vector_t v = bno055_getVectorEuler();
printf("Heading: %.2f Roll: %.2f Pitch: %.2f\r\n", v.x, v.y, v.z);
HAL_Delay(100);
```
- Add to USER CODE BEGIN 4. This allows prints to the console for debugging.
```C
int __io_putchar(int ch) {
//	HAL_UART_Transmit(&huart2, (uint8_t*) &ch, 1, HAL_MAX_DELAY); // Transmit the character over UART
	ITM_SendChar(ch); // Send the character to the SWO console (for debugging)
	return ch;
}
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
printf("Status: %d \r\n", bno055_getSystemStatus());
```



## System Error Codes

These can be ignored if System Status is 0.

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
printf("Status: %d \r\n", bno055_getSystemStatus()); // if this is zero, ignore any system error
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

## Calibration

Read 3.11 of the BNO055 datasheet for proper calibration.

The sensor auto calibrates on start up everytime, if you do care about accuracy follow the example below.
Note: This is my example, I'm not 100% certain this is the best way or proper way. Just a method that works for me.

Calibration States:
| Code | Description    |
|------|----------------|
| 0    | Uncalibrated   |
| 1    | Partially      |
| 2    | Mostly         |
| 3    | Calibrated     |


Use the following code:

```C
/* USER CODE BEGIN PV */
bno055_calibration_data_t savedCalData = {
		.offset.accel = { .x = -7, .y = 6, .z =-33 },
		.offset.mag = { .x = -76, .y = -423, .z = -216 },
		.offset.gyro = { .x = -2, .y = -3, .z = -1 },
		.radius.accel = 1000,
		.radius.mag = 1484
};
/* USER CODE END PV */
```

```C
/* USER CODE BEGIN 2 */
bno055_assignI2C(&hi2c1); // Assign the I2C handle to the BNO055 library
bno055_setup();
bno055_setOperationMode(BNO055_OPERATION_MODE_NDOF);
//	bno055_setCalibrationData(savedCalData); // uncomment this to set calibrated data.

bno055_printUnits();
HAL_Delay(1000);

bno055_runCalibration();
/* USER CODE END 2 */
```

This code is used to collect the calibration data. It will stop and hold once the calibration levels reach 3 for all.

- For gyroscope: keep it still.
- For acceleration: move it in 45 degree increments.
- For magnometer: move the advice in a 8 figure.

Once the program has finished and printed the results. Write them in `savedCalData` 

After add `bno055_setCalibrationData(savedCalData);`


# Full Working Example
```C
/* USER CODE BEGIN Header */
/**
 ******************************************************************************
 * @file           : main.c
 * @brief          : Main program body
 ******************************************************************************
 * @attention
 *
 * Copyright (c) 2026 STMicroelectronics.
 * All rights reserved.
 *
 * This software is licensed under terms that can be found in the LICENSE file
 * in the root directory of this software component.
 * If no LICENSE file comes with this software, it is provided AS-IS.
 *
 ******************************************************************************
 */
/* USER CODE END Header */
/* Includes ------------------------------------------------------------------*/
#include "main.h"

/* Private includes ----------------------------------------------------------*/
/* USER CODE BEGIN Includes */
#include "bno055_stm32.h"
#include <stdio.h>
/* USER CODE END Includes */

/* Private typedef -----------------------------------------------------------*/
/* USER CODE BEGIN PTD */

/* USER CODE END PTD */

/* Private define ------------------------------------------------------------*/
/* USER CODE BEGIN PD */

/* USER CODE END PD */

/* Private macro -------------------------------------------------------------*/
/* USER CODE BEGIN PM */

/* USER CODE END PM */

/* Private variables ---------------------------------------------------------*/
I2C_HandleTypeDef hi2c1;

UART_HandleTypeDef huart2;

/* USER CODE BEGIN PV */
bno055_vector_t v; // Create an instance of the BNO055 sensor structure
int8_t temperature; // Variable to hold the temperature reading

bno055_calibration_data_t savedCalData = {
		.offset.accel = { .x = -7, .y = 6, .z =-33 },
		.offset.mag = { .x = -76, .y = -423, .z = -216 },
		.offset.gyro = { .x = -2, .y = -3, .z = -1 },
		.radius.accel = 1000,
		.radius.mag = 1484
};
/* USER CODE END PV */

/* Private function prototypes -----------------------------------------------*/
void SystemClock_Config(void);
static void MX_GPIO_Init(void);
static void MX_USART2_UART_Init(void);
static void MX_I2C1_Init(void);
/* USER CODE BEGIN PFP */

/* USER CODE END PFP */

/* Private user code ---------------------------------------------------------*/
/* USER CODE BEGIN 0 */

/* USER CODE END 0 */

/**
 * @brief  The application entry point.
 * @retval int
 */
int main(void) {

	/* USER CODE BEGIN 1 */

	/* USER CODE END 1 */

	/* MCU Configuration--------------------------------------------------------*/

	/* Reset of all peripherals, Initializes the Flash interface and the Systick. */
	HAL_Init();

	/* USER CODE BEGIN Init */

	/* USER CODE END Init */

	/* Configure the system clock */
	SystemClock_Config();

	/* USER CODE BEGIN SysInit */

	/* USER CODE END SysInit */

	/* Initialize all configured peripherals */
	MX_GPIO_Init();
	MX_USART2_UART_Init();
	MX_I2C1_Init();
	/* USER CODE BEGIN 2 */
	bno055_assignI2C(&hi2c1); // Assign the I2C handle to the BNO055 library
	bno055_setup();
	bno055_setOperationMode(BNO055_OPERATION_MODE_NDOF);
	bno055_setCalibrationData(savedCalData);

//	bno055_printUnits();
	HAL_Delay(1000);

//	bno055_runCalibration();

	/* USER CODE END 2 */

	/* Infinite loop */
	/* USER CODE BEGIN WHILE */
	while (1) {
		/* USER CODE END WHILE */

		/* USER CODE BEGIN 3 */
		v = bno055_getVectorEuler();
		printf("Euler: Heading=%0.2f, Roll=%0.2f, Pitch=%0.2f\r\n", v.x, v.y, v.z);
		v = bno055_getVectorAccelerometer();
		printf("Accel: X=%0.2f m/s^2, Y=%0.2f m/s^2, Z=%0.2f m/s^2\r\n", v.x, v.y, v.z);
		v = bno055_getVectorGyroscope();
		printf("Gyro: X=%0.2f dps, Y=%0.2f dps, Z=%0.2f dps\r\n", v.x, v.y, v.z);
		v = bno055_getVectorMagnetometer();
		printf("Mag: X=%0.2f uT, Y=%0.2f uT, Z=%0.2f uT\r\n", v.x, v.y, v.z);
		temperature = bno055_getTemp();
		printf("Temperature: %d C\r\n", temperature);
		printf("\r\n");
		HAL_Delay(1000);
	}
	/* USER CODE END 3 */
}

/**
 * @brief System Clock Configuration
 * @retval None
 */
void SystemClock_Config(void) {
	RCC_OscInitTypeDef RCC_OscInitStruct = { 0 };
	RCC_ClkInitTypeDef RCC_ClkInitStruct = { 0 };

	/** Configure the main internal regulator output voltage
	 */
	__HAL_RCC_PWR_CLK_ENABLE();
	__HAL_PWR_VOLTAGESCALING_CONFIG(PWR_REGULATOR_VOLTAGE_SCALE1);

	/** Initializes the RCC Oscillators according to the specified parameters
	 * in the RCC_OscInitTypeDef structure.
	 */
	RCC_OscInitStruct.OscillatorType = RCC_OSCILLATORTYPE_HSI;
	RCC_OscInitStruct.HSIState = RCC_HSI_ON;
	RCC_OscInitStruct.HSICalibrationValue = RCC_HSICALIBRATION_DEFAULT;
	RCC_OscInitStruct.PLL.PLLState = RCC_PLL_ON;
	RCC_OscInitStruct.PLL.PLLSource = RCC_PLLSOURCE_HSI;
	RCC_OscInitStruct.PLL.PLLM = 8;
	RCC_OscInitStruct.PLL.PLLN = 180;
	RCC_OscInitStruct.PLL.PLLP = RCC_PLLP_DIV2;
	RCC_OscInitStruct.PLL.PLLQ = 2;
	RCC_OscInitStruct.PLL.PLLR = 2;
	if (HAL_RCC_OscConfig(&RCC_OscInitStruct) != HAL_OK) {
		Error_Handler();
	}

	/** Activate the Over-Drive mode
	 */
	if (HAL_PWREx_EnableOverDrive() != HAL_OK) {
		Error_Handler();
	}

	/** Initializes the CPU, AHB and APB buses clocks
	 */
	RCC_ClkInitStruct.ClockType = RCC_CLOCKTYPE_HCLK | RCC_CLOCKTYPE_SYSCLK
			| RCC_CLOCKTYPE_PCLK1 | RCC_CLOCKTYPE_PCLK2;
	RCC_ClkInitStruct.SYSCLKSource = RCC_SYSCLKSOURCE_PLLCLK;
	RCC_ClkInitStruct.AHBCLKDivider = RCC_SYSCLK_DIV1;
	RCC_ClkInitStruct.APB1CLKDivider = RCC_HCLK_DIV4;
	RCC_ClkInitStruct.APB2CLKDivider = RCC_HCLK_DIV2;

	if (HAL_RCC_ClockConfig(&RCC_ClkInitStruct, FLASH_LATENCY_5) != HAL_OK) {
		Error_Handler();
	}
}

/**
 * @brief I2C1 Initialization Function
 * @param None
 * @retval None
 */
static void MX_I2C1_Init(void) {

	/* USER CODE BEGIN I2C1_Init 0 */

	/* USER CODE END I2C1_Init 0 */

	/* USER CODE BEGIN I2C1_Init 1 */

	/* USER CODE END I2C1_Init 1 */
	hi2c1.Instance = I2C1;
	hi2c1.Init.ClockSpeed = 100000;
	hi2c1.Init.DutyCycle = I2C_DUTYCYCLE_2;
	hi2c1.Init.OwnAddress1 = 0;
	hi2c1.Init.AddressingMode = I2C_ADDRESSINGMODE_7BIT;
	hi2c1.Init.DualAddressMode = I2C_DUALADDRESS_DISABLE;
	hi2c1.Init.OwnAddress2 = 0;
	hi2c1.Init.GeneralCallMode = I2C_GENERALCALL_DISABLE;
	hi2c1.Init.NoStretchMode = I2C_NOSTRETCH_DISABLE;
	if (HAL_I2C_Init(&hi2c1) != HAL_OK) {
		Error_Handler();
	}
	/* USER CODE BEGIN I2C1_Init 2 */

	/* USER CODE END I2C1_Init 2 */

}

/**
 * @brief USART2 Initialization Function
 * @param None
 * @retval None
 */
static void MX_USART2_UART_Init(void) {

	/* USER CODE BEGIN USART2_Init 0 */

	/* USER CODE END USART2_Init 0 */

	/* USER CODE BEGIN USART2_Init 1 */

	/* USER CODE END USART2_Init 1 */
	huart2.Instance = USART2;
	huart2.Init.BaudRate = 115200;
	huart2.Init.WordLength = UART_WORDLENGTH_8B;
	huart2.Init.StopBits = UART_STOPBITS_1;
	huart2.Init.Parity = UART_PARITY_NONE;
	huart2.Init.Mode = UART_MODE_TX_RX;
	huart2.Init.HwFlowCtl = UART_HWCONTROL_NONE;
	huart2.Init.OverSampling = UART_OVERSAMPLING_16;
	if (HAL_UART_Init(&huart2) != HAL_OK) {
		Error_Handler();
	}
	/* USER CODE BEGIN USART2_Init 2 */

	/* USER CODE END USART2_Init 2 */

}

/**
 * @brief GPIO Initialization Function
 * @param None
 * @retval None
 */
static void MX_GPIO_Init(void) {
	GPIO_InitTypeDef GPIO_InitStruct = { 0 };
	/* USER CODE BEGIN MX_GPIO_Init_1 */

	/* USER CODE END MX_GPIO_Init_1 */

	/* GPIO Ports Clock Enable */
	__HAL_RCC_GPIOC_CLK_ENABLE();
	__HAL_RCC_GPIOH_CLK_ENABLE();
	__HAL_RCC_GPIOA_CLK_ENABLE();
	__HAL_RCC_GPIOB_CLK_ENABLE();

	/*Configure GPIO pin Output Level */
	HAL_GPIO_WritePin(LD2_GPIO_Port, LD2_Pin, GPIO_PIN_RESET);

	/*Configure GPIO pin : B1_Pin */
	GPIO_InitStruct.Pin = B1_Pin;
	GPIO_InitStruct.Mode = GPIO_MODE_IT_FALLING;
	GPIO_InitStruct.Pull = GPIO_NOPULL;
	HAL_GPIO_Init(B1_GPIO_Port, &GPIO_InitStruct);

	/*Configure GPIO pin : LD2_Pin */
	GPIO_InitStruct.Pin = LD2_Pin;
	GPIO_InitStruct.Mode = GPIO_MODE_OUTPUT_PP;
	GPIO_InitStruct.Pull = GPIO_NOPULL;
	GPIO_InitStruct.Speed = GPIO_SPEED_FREQ_LOW;
	HAL_GPIO_Init(LD2_GPIO_Port, &GPIO_InitStruct);

	/* USER CODE BEGIN MX_GPIO_Init_2 */

	/* USER CODE END MX_GPIO_Init_2 */
}

/* USER CODE BEGIN 4 */
int __io_putchar(int ch) {
//	HAL_UART_Transmit(&huart2, (uint8_t*) &ch, 1, HAL_MAX_DELAY); // Transmit the character over UART
	ITM_SendChar(ch); // Send the character to the SWO console (for debugging)
	return ch;
}
/* USER CODE END 4 */

/**
 * @brief  This function is executed in case of error occurrence.
 * @retval None
 */
void Error_Handler(void) {
	/* USER CODE BEGIN Error_Handler_Debug */
	/* User can add his own implementation to report the HAL error return state */
	__disable_irq();
	while (1) {
	}
	/* USER CODE END Error_Handler_Debug */
}
#ifdef USE_FULL_ASSERT
/**
  * @brief  Reports the name of the source file and the source line number
  *         where the assert_param error has occurred.
  * @param  file: pointer to the source file name
  * @param  line: assert_param error line source number
  * @retval None
  */
void assert_failed(uint8_t *file, uint32_t line)
{
  /* USER CODE BEGIN 6 */
  /* User can add his own implementation to report the file name and line number,
     ex: printf("Wrong parameters value: file %s on line %d\r\n", file, line) */
  /* USER CODE END 6 */
}
#endif /* USE_FULL_ASSERT */

```

