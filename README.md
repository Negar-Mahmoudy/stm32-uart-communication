# STM32 UART Communication with PC (Polling Mode)

This project is about how to establish **serial communication (UART)** between an STM32 microcontroller and a PC. The PC is connected via a serial interface (ESP board). The STM32 board sends and receives commands to control an LED.

---

## Project Description
The STM32 board communicates with the laptop over UART and an **ESP board (with RST tied to GND to disable MCU)** was used as a USB-to-Serial bridge.  
    
```

Enter command for LED (led0 or led1):

````
The user then types a command in **RealTerm** (or any serial terminal) and STM32 checks the input:  
- `led0` → LED turns **OFF**.  
- `led1` → LED turns **ON**.  
then a confirmation message is sent back to the PC.
- 
![REALTERM OUTPUT](https://github.com/Negar-Mahmoudy/stm32-uart-communication/blob/main/images/uart.png?raw=true)

---

## Configuration
- **USART1**:  
- Baud rate: `115200`  
- Word length: `8 bits`  
- Stop bits: `1`  
- Parity: `None`  
- Mode: `TX/RX`  
- **GPIO**:  
- `RED_LED_Pin` → Configured as output, toggled based on received commands.  
- **Connection**:  
- `TX (STM32)` → `TX (ESP)`  
- `RX (STM32)` → `RX (ESP)`  
- `GND` shared between STM32 and ESP.  
- **Software**: RealTerm(or any serial terminal).  

---

## Code Explanation
### Transmitting
```c
const char *msg = "\n\rEnter command for LED (led0 or led1):\n\r";
HAL_UART_Transmit(&huart1, (uint8_t *)msg, strlen(msg), 10);
````

Message is stored in **Flash** (`const char*`). and is Sent to the PC through UART.

### Receiving

```c
HAL_UART_Receive(&huart1, (uint8_t *)rxBuffer, RX_BUFFER_SIZE, 5000);
```

Waits for up to `5000 ms` for input and stores up to `RX_BUFFER_SIZE` characters.

### Acting on commands

```c
if(strncmp(rxBuffer, "led0", 4) == 0) {
    HAL_GPIO_WritePin(RED_LED_GPIO_Port, RED_LED_Pin, GPIO_PIN_RESET);
    HAL_UART_Transmit(&huart1, (uint8_t *)"\nLED is off.", 13, 10);
}
else if(strncmp(rxBuffer, "led1", 4) == 0) {
    HAL_GPIO_WritePin(RED_LED_GPIO_Port, RED_LED_Pin, GPIO_PIN_SET);
    HAL_UART_Transmit(&huart1, (uint8_t *)"\nLED is on.", 12, 10);
}
```

Compares the received string with predefined commands then updates LED state and sends feedback.
