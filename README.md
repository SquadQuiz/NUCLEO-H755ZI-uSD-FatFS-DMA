# NUCLEO-H755ZI-Q [uSD + FatFs + DMA]

## Pre-requisites

- Hardware
  - NUCLEO-H755ZI-Q (STM32H755ZI Development board): [Click!](https://www.st.com/en/evaluation-tools/nucleo-h755zi-q.html)
  ![NUCLEO-H755ZI-Q](/docs/assets/nucleo-h755zi-q-bg.jpeg)
  - Digilent Pmod MicroSD (4-bit SDIO Breakout board): [Click!](https://digilent.com/reference/pmod/pmodmicrosd/start)
  ![Pmod-MicroSD](/docs/assets/pmodmicrosd-oblique.png)

  - Pinout

    ![J1-Conn](/docs/assets/pmod-j1conn.png)

    Connector J1- Pin Descriptions

    | Pin | Signal | Description         | Pin | Signal | Description         |
    |-----|--------|---------------------|-----|--------|---------------------|
    | 1   | DAT3   | Chip Select / Data3 | 7   | DAT1   | Data 1              |
    | 2   | CMD    | MOSI / Command      | 8   | DAT2   | Data 2              |
    | 3   | DAT0   | MISO / Data0        | 9   | CD     | Card Detect         |
    | 4   | CK     | Serial Clock        | 10  | NC     | Not Connected       |
    | 5   | GND    | Power Supply Ground | 11  | GND    | Power Supply Ground |
    | 6   | VCC    | Power Supply (3.3V) | 12  | VCC    | Power Supply (3.3V) |

- Software
  - IDE
    - STM32CubeIDE
  - Drivers
    - STM32 HAL H7xx Drivers Package
  - Middlewares
    - FatFS (File System Format)

## NUCLEO-H755ZI-Q SDMMC1 I/F Pinout

- SDMMC1 Interface
  - Connector CN8 --SDMMC-- (Arduino Uno compatible)

    ![CN8-SH](/docs/assets/cn8-sdmmc1.jpeg)

  - Schematics

    ![CN8-SDMMC-SCH](/docs/assets/sch-nucleo-cn8.png)

  - SDMMC1 Pinout Table

    | Pin | Pin name | Pin Assignment | Pin Context Assignment | User Label        |
    | --- | -------- | -------------- | ---------------------- | ----------------- |
    | 37  | PA0      | GPIO_INPUT     | Cortex-M7              | SDMMC1_uSD_DETECT |
    | 114 | PD2      | SDMMC1         | Cortex-M7              | SDMMC1_CMD        |
    | 95  | PC8      | SDMMC1         | Cortex-M7              | SDMMC1_D0         |
    | 96  | PC9      | SDMMC1         | Cortex-M7              | SDMMC1_D1         |
    | 109 | PC10     | SDMMC1         | Cortex-M7              | SDMMC1_D2         |
    | 110 | PC11     | SDMMC1         | Cortex-M7              | SDMMC1_D3         |
    | 111 | PC12     | SDMMC1         | Cortex-M7              | SDMMC1_CK         |

## Steps

1. Creating a STM32 project with STM32CubeMX,  
  New Project -> Board Selector -> NUCLEO-H755ZI-Q  
  Then Click "__Start Project__"  

    ![New-project](/docs/cubemx/newproj-h755.png)

2. Select components from BSP Drivers for NUCLEO-H755ZI-Q  

    ![bsp-select](/docs/cubemx/bsp-select.png)

3. CubeMX -> Connectivity -> Enable SDMMC1 on context of Cortex-M7

    ![sd-conf](/docs/cubemx/sd-conf.png)

      - SDMMC Mode:
        - "__SD 4 bits Wide Bus mode__"

      - SDMMC1 -> Parameter Settings
        - SDMMC1 Clock divide factor = 0x04 or __SDMMC_NSPEED_CLK_DIV__

        ```c
        // In .../stm32h7xx_ll_sdmmc.h

        /* SDMMC Initialization Frequency (400KHz max) for Peripheral CLK 200MHz*/
        #define SDMMC_INIT_CLK_DIV ((uint8_t)0xFA)

        /* SDMMC Default Speed Frequency (25Mhz max) for Peripheral CLK 200MHz*/
        #define SDMMC_NSPEED_CLK_DIV ((uint8_t)0x4)

        /* SDMMC High Speed Frequency (50Mhz max) for Peripheral CLK 200MHz*/
        #define SDMMC_HSPEED_CLK_DIV ((uint8_t)0x2)
        ```

      - SDMMC1 -> NVIC Settings

        ![sdmmc-isr](/docs/cubemx/sdmmc-isr.png)

        - Enabled SDMMC1 Global Interrupt, you change ISR priority on NVIC page

4. Clock Configuration (RCC), Default we will use HSI Oscillator

    ![alt text](/docs/cubemx/rcc-clock.png)
    - Clock Settings
      - SYSCLK = 400 MHz
      - CPU1_CLK (M7) = 400 MHz
      - CPU2_CLK (M4) = 200 MHz
      - PCLK = 200 MHz

    - SDMMC1, Clock Mux

      ![alt text](/docs/cubemx/sdmmc-clk.png)

      - SDMMC1_CLK = 200 MHz
      - Actual SDMMC1 Clock = SDMMC1_CLK / SDMMC_DIVIDE_FACTOR
      - ACTUAL_SDMMC1_CLK = (200 MHz/ 4) = 50 MHz

5. FATFS Middleware Configuration
    - Middleware and Software Packs -> FATFS_M7
      - Enabled FATFS_M7 with default defines configuration and select it as "__SD CARD__"

      ![alt text](/docs/cubemx/fatfs-conf.png)

    - FATFS_M7 -> Advanced Settings, Enabled DMA Template

      ![alt text](/docs/cubemx/fatfs-dma.png)

      - FATFS_M7 -> Platform Settings

        - Select PA0[SDMMC1_uSD_DETECT] as SD Card Detect pin

        ![alt text](/docs/cubemx/fatfs-platform.png)

6. MPU Configuration

    - Memory Protection Unit (MPU)

      - MPU Region 0 and CPU L1 Cache

        ![MPU-Region0](/docs/cubemx/mpu-region0.png)
  
        - Speculation default mode
          - Enabled
        - Cortex Interface Settings
          - CPU I-Cache
            - Enabled
          - CPU D-Cache
            - Enabled
        - For MPU Region 0
          - Keep everything as default

    - MPU Region 1 "__AXIM SRAM Region__"

      ![MPU-Region1](/docs/cubemx/mpu-region1.png)

      - For MPU Region 1
        - MPU Regin
          - Enabled
        - MPU Region Base Address
          - 0x24000000 (AXIM SRAM)
        - MPU Region Size
          - 512 KB
        - MPU Region SubRegion Disable
          - 0x0
        - MPU Region TEX field level
          - Level 1
        - MPU Region Access Permission
          - ALL ACCESS PERMISSION
        - MPU Region Instruction Access
          - ENABLE
        - MPU Region Shareability Permission
          - DISABLE
        - MPU Region Cacheable Permission
          - DISABLE
        - MPU Region Bufferable Permission
          - DISABLE

      - STM32H755xx Memory Map
        - ![axim-map](/docs/datasheet/memory-map-ram.png)

7. NVIC (Nested Vector Interrupt Controller)

    ![alt text](/docs/cubemx/nvic-conf.png)
      - ISR Priority
        - Time base: Systick Timer = 14
        - SDMMC1 = 14
        - EXTI line[15:10] = 15

8. Linker settings

    ![linker-conf](/docs/cubemx/linker-conf.png)

      - Cortex-M7
        - Minimum Heap Size = 0x400
        - Minimum Stack Size = 0x800
      - Cortex-M4
        - Minimum Heap Size = 0x400
        - Minimum Stack Size = 0x800

9. Add source code to ../CM7/Core/main.c

```c
  /* Private function prototypes -----------------------------------------------*/
  /* USER CODE BEGIN PFP */

  static void FS_FormatDisk(void);
  static void FS_FileOperations(void);
  static uint8_t Buffercmp(uint8_t* pBuffer1, uint8_t* pBuffer2, uint32_t BufferLength);

  /* USER CODE END PFP */

  /* Private user code ---------------------------------------------------------*/
  /* USER CODE BEGIN 0 */

  uint8_t workBuffer[_MAX_SS];

  /*
  * ensure that the read buffer 'rtext' is 32-Byte aligned in address and size
  * to guarantee that any other data in the cache won't be affected when the 'rtext'
  * is being invalidated.
  */
  ALIGN_32BYTES(uint8_t rtext[64]);

  /* USER CODE END 0 */

  int main(void)
  {
    /* USER CODE BEGIN BSP */
    /* -- Sample board code to send message over COM1 port ---- */
    printf("[CORE_CM7]: Program Starting... \r\n");

    BSP_LED_On(LED_YELLOW);

  //  FS_FormatDisk();
    FS_FileOperations();

    BSP_LED_Off(LED_YELLOW);

    /* USER CODE END BSP */
  }

  /* USER CODE BEGIN 4 */

  /**
   * @brief Format the disk with the FatFs file system.
   * 
   * This function mounts the file system, formats the SD card,
   * and provides diagnostic prints for each step. If any operation
   * fails, it calls Error_Handler().
   * 
   * @note Ensure that the file system is properly mounted and the
   *       SD card is inserted before calling this function.
   */
  static void FS_FormatDisk(void)
  {
    FRESULT fres; /* FatFs function common result code */

    /* Mount the file system */
    fres = f_mount(&SDFatFS, (TCHAR const*)SDPath, 0);
    if (fres == FR_OK)
    {
      printf("[CORE_CM7/FatFs]: Successfully mounted SD Card\n");

      /* Format the file system */
      fres = f_mkfs((TCHAR const*)SDPath, FM_ANY, 0, workBuffer, sizeof(workBuffer));
      if (fres == FR_OK)
      {
        printf("[CORE_CM7/FatFs]: Successfully formatted SD Card\n");
        return;
      }
    }

    /* Error handling */
    Error_Handler();
  }

  /**
   * @brief Performs file operations on an SD card using the FatFs library.
   *
   * This function mounts the file system, creates and opens a text file for writing,
   * writes data to the file, closes the file, then reopens it for reading, and finally
   * reads the data back. It compares the read data with the written data to ensure
   * the integrity of the file operations. If any operation fails, it calls the 
   * Error_Handler() function.
   *
   * @note Ensure that the file system is properly mounted and the
   *       SD card is inserted before calling this function.
   *
   * @return void
   */
  static void FS_FileOperations(void)
  {
    FRESULT res;                      /* FatFs function common result code */
    uint32_t bytesWritten, bytesRead; /* File write/read counts */
    uint8_t wtext[] = "[CORE_CM7]: This is STM32 working with FatFs + DMA"; /* File write buffer */

    /* Mount the file system */
    res = f_mount(&SDFatFS, (TCHAR const*)SDPath, 0);
    if (res != FR_OK)
    {
      printf("[CORE_CM7/FatFs]: Failed to mount SD card (error code: %d)\n", res);
      Error_Handler();
    }
    else
    {
      printf("[CORE_CM7/FatFs]: Successfully mounted SD Card\n");

      /* Open or create a new text file for writing */
      res = f_open(&SDFile, "STM32.TXT", FA_CREATE_ALWAYS | FA_WRITE);
      if (res != FR_OK)
      {
        printf("[CORE_CM7/FatFs]: Failed to open file for writing (error code: %d)\n", res);
        Error_Handler();
      }
      else
      {
        printf("[CORE_CM7/FatFs]: Successfully opened file for writing\n");

        /* Write data to the text file */
        res = f_write(&SDFile, wtext, sizeof(wtext), (void*)&bytesWritten);
        if (res != FR_OK || bytesWritten != sizeof(wtext))
        {
          printf("[CORE_CM7/FatFs]: Failed to write data to file (error code: %d, bytes "
                "written: %lu)\n",
                res, bytesWritten);
          Error_Handler();
        }
        else
        {
          printf("[CORE_CM7/FatFs]: Data successfully written to file (bytes written: %lu)\n",
                bytesWritten);

          /* Close the text file after writing */
          f_close(&SDFile);

          /* Open the text file for reading */
          res = f_open(&SDFile, "STM32.TXT", FA_READ);
          if (res != FR_OK)
          {
            printf("[CORE_CM7/FatFs]: Failed to open file for reading (error code: %d)\n", res);
            Error_Handler();
          }
          else
          {
            printf("[CORE_CM7/FatFs]: Successfully opened file for reading\n");

            /* Read data from the text file */
            res = f_read(&SDFile, rtext, sizeof(wtext), (void*)&bytesRead);
            if (res != FR_OK || bytesRead != bytesWritten)
            {
              printf("[CORE_CM7/FatFs]: Failed to read the correct amount of data (error code: %d, "
                    "bytes read: %lu, expected: %lu)\n",
                    res, bytesRead, bytesWritten);
              Error_Handler();
            }
            else
            {
              printf("[CORE_CM7/FatFs]: Data successfully read from file (bytes read: %lu)\n",
                    bytesRead);

              /* Close the text file after reading */
              f_close(&SDFile);

              /* Compare read data with the written data */
              if (Buffercmp(rtext, wtext, bytesWritten) == 0)
              {
                printf("[CORE_CM7/FatFs]: Data written and read are identical\n");
                BSP_LED_On(LED_GREEN);
                return;
              }
              else
              {
                printf("[CORE_CM7/FatFs]: Data mismatch\n");
                Error_Handler();
              }
            }
          }
        }
      }
    }

    /* If any error occurs, handle it here */
    Error_Handler();
  }

  /**
   * @brief  Compares two buffers.
   * @param  pBuffer1, pBuffer2: buffers to be compared.
   * @param  BufferLength: buffer's length
   * @retval 1: pBuffer identical to pBuffer1
   *         0: pBuffer differs from pBuffer1
   */
  static uint8_t Buffercmp(uint8_t* pBuffer1, uint8_t* pBuffer2, uint32_t BufferLength)
  {
    while (BufferLength--)
    {
      if (*pBuffer1 != *pBuffer2)
      {
        return 1;
      }

      pBuffer1++;
      pBuffer2++;
    }
    return 0;
  }

  /* USER CODE END 4 */
```

## Demonstration

![demo](/docs//assets/demo.png)

## Reference

- Datasheet:
  - [STM32H755xI Dual 32-bit Arm® Cortex®-M7 up to 480MHz and -M4 MCUs, 2MB flash, 1MB RAM, 46 com. and analog interfaces, SMPS, crypto](https://www.st.com/resource/en/datasheet/stm32h755zi.pdf)
- Reference manual:
  - [STM32H745/755 and STM32H747/757 advanced Arm®-based 32-bit MCUs](https://www.st.com/resource/en/reference_manual/rm0399-stm32h745755-and-stm32h747757-advanced-armbased-32bit-mcus-stmicroelectronics.pdf)
  - Snapshot:
    - [Block-Diagram](/docs/datasheet/STM32H755xx-Block-Diagram.pdf)
    - [Memory-Map](/docs/datasheet/STM32H755xx-Memory-Map.pdf)
    - [Bus-Matrix](/docs/datasheet/STM32H755xx-Bus-Matrix.pdf)
- Schematics
  - [MB1363-H755ZIQ-D01 Schematics](https://www.st.com/resource/en/schematic_pack/mb1363-h755ziq-d01_schematic.pdf)
- Application Note
  - [AN5361 Getting started with projects based on dual-core STM32H7 microcontrollers in STM32CubeIDE](https://www.st.com/resource/en/application_note/an5361-getting-started-with-projects-based-on-dualcore-stm32h7-microcontrollers-in-stm32cubeide-stmicroelectronics.pdf)
  - [AN5286 STM32H7x5/x7 dual-core microcontroller debugging](https://www.st.com/content/ccc/resource/technical/document/application_note/group1/96/bf/35/36/e6/97/42/2e/DM00597308/files/DM00597308.pdf/jcr:content/translations/en.DM00597308.pdf)
  - [AN5200 Getting started with STM32H7 MCUs SDMMC host controller](https://www.st.com/resource/en/application_note/dm00525510-getting-started-with-stm32h7-series-sdmmc-host-controller-stmicroelectronics.pdf)
  - [AN4838 Introduction to memory protection unit management on STM32 MCUs](https://www.st.com/resource/en/application_note/an4838-introduction-to-memory-protection-unit-management-on-stm32-mcus-stmicroelectronics.pdf)
  - [AN4839 Level 1 cache on STM32F7 Series and STM32H7 Series](https://www.st.com/resource/en/application_note/an4839-level-1-cache-on-stm32f7-series-and-stm32h7-series-stmicroelectronics.pdf)
- User manual
  - [UM1721 Developing applications on STM32Cube™ with FatFs](https://www.st.com/resource/en/user_manual/um1721-developing-applications-on-stm32cube-with-fatfs-stmicroelectronics.pdf)
