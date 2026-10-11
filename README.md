Part 1: Install the tools (once)

STM32CubeIDE. Download it from st.com (STM32CubeIDE page) and install it with the defaults. It includes the compiler, the debugger and the ST-LINK driver. ST may ask you to sign in with a free myST account to download.

STM32CubeMX. Newer CubeIDE versions include it. If yours doesn’t, install STM32CubeMX from st.com too. It is only used to download the firmware package.
The STM32H7 firmware package (STM32Cube FW_H7).

Open STM32CubeMX and go to Help → Manage embedded software packages.

Open the STM32H7 section, tick STM32Cube MCU Package for STM32H7 Series, version 1.13.0, and click Install.

This package contains the example, the drivers and the LCD code, so there is nothing else to install.

Hardware. 
Connect the ST-LINK USB port of the STM32H743I-EVAL board to the PC and power the board. Windows should recognize the ST-LINK automatically.

Steps in CubeMX

File → New Project → Board Selector → STM32H743I-EVAL2, then click Start Project. Answer Yes to default initialization.

In System Core → GPIO, check the two pins:

TAMPER (PC13): mode GPIO_Input, Pull-up.

LED1_RGB [LD1_Green] (PF10): mode Output Push Pull, no pull, initial level Low.

Check that RTC has no green tick, so it doesn’t take over PC13.

In Project Manager, set the name Tamper_Test, choose a short folder path, set Toolchain / IDE to STM32CubeIDE, and click GENERATE CODE.

In CubeIDE, use File → Import → STM32CubeMX/STM32CubeIDE Project and select the project folder.

Open Core → Inc → main.h to find the pin names. They are TAMPER_Pin, TAMPER_GPIO_Port, LED1_RGB_Pin and LED1_RGB_GPIO_Port. The code (in Core/Src/main.c) Add these two lines in the USER CODE BEGIN PD section:

Part 2: Open the example

In CubeIDE, go to File → Import → General → Existing Projects into Workspace → Next.

Click Browse and find the firmware package folder. If you don’t know where it is, open CubeMX, go to Help → Updater Settings, and read the Repository Folder.

Inside the package, open Projects → STM32H743I-EVAL → Examples → JPEG → JPEG_DecodingFromFLASH_DMA → STM32CubeIDE and select that folder.

Tick the project, untick “Copy projects into workspace”, and click Finish.

Don’t move or rename the firmware package afterwards, or the project stops building.

Part 3: 
Add the image and edit the code



Part 3: Add the image and edit the code

Drag display_image.h (the image file you share with them) onto the folder in Project Explorer that contains main.h, and choose Copy files.

i have uploaded display_image.h file in repositry ,you can access by using that,in that you just convert you image into byte by using online free website,and copy that byte array for display_image.h.

Edit 1: 
change
#include "image_320_240_jpg.h" to
c
#include "display_image.h"


Edit 2: replace the decode call after LCD_BriefDisplay(); with

c
  __HAL_RCC_GPIOC_CLK_ENABLE();
  GPIO_InitTypeDef tamper = {0};
  tamper.Pin  = GPIO_PIN_13;
  tamper.Mode = GPIO_MODE_INPUT;
  tamper.Pull = GPIO_PULLUP;
  HAL_GPIO_Init(GPIOC, &tamper);

  while (HAL_GPIO_ReadPin(GPIOC, GPIO_PIN_13) != GPIO_PIN_RESET)
  {
  }

  JPEG_Decode_DMA(&JPEG_Handle, (uint32_t)display_jpeg, DISPLAY_JPEG_SIZE, JPEG_OUTPUT_DATA_BUFFER);

Edit 3: replace the xPos and yPos lines with

c
  if ((LCD_X_Size < JPEG_Info.ImageWidth) || (LCD_Y_Size < JPEG_Info.ImageHeight))
  {
    Error_Handler();
  }
  xPos = (LCD_X_Size - JPEG_Info.ImageWidth)/2;
  yPos = (LCD_Y_Size - JPEG_Info.ImageHeight)/2;

Edit 4: in DMA2D_CopyBuffer, change LCD_Y_Size to LCD_X_Size

c
  destination = (uint32_t)pDst + ((y * LCD_X_Size) + x) * 4;

  
Part 4: Build, run and test

Press Ctrl+S, then Project → Clean, then click the hammer icon. You want 0 errors.

Select the project, then use bug icon arrow → Debug As → STM32 C/C++ Application → OK.

When it stops at main(), press F8.

The LCD shows the blue ST header. Press the Tamper button, and the image appears.


