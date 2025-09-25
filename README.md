# STM32 UART Communication with PC (Polling Mode)

This project demonstrates how to establish **serial communication (UART)** between an STM32 microcontroller and a PC. The PC is connected via a serial interface (e.g., ESP board or Arduino used as USB-to-Serial converter). The STM32 board sends and receives commands to control an LED.

---

## Project Goal
The main purpose of this project is to practice:  
- Setting up **UART communication** on STM32 using HAL.  
- Sending and receiving data in **polling mode**.  
- Understanding the use of `const char*` for storing constant strings in Flash memory.  
- Creating a simple **command interface**: user can type `led0` or `led1` to turn an LED OFF or ON.  

---

## Project Description
- The STM32 board communicates with the laptop over UART (USART1 @ 115200 baud).  
- An **ESP board (with RST tied to GND to disable MCU)** was used as a USB-to-Serial bridge.  
- The program first transmits a message to the PC:  
```

Enter command for LED (led0 or led1):

````
- The user then types a command in **RealTerm** (or any serial terminal).  
- STM32 parses the input:  
- `led0` → LED turns **OFF**.  
- `led1` → LED turns **ON**.  
- A confirmation message is sent back to the PC.
- 
![REALTERM OUTPUT](images/uart_diagram.png)

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
### Transmitting a message
```c
const char *msg = "\n\rEnter command for LED (led0 or led1):\n\r";
HAL_UART_Transmit(&huart1, (uint8_t *)msg, strlen(msg), 10);
````

* Message is stored in **Flash** (`const char*`).
* Sent to the PC through UART.

### Receiving user input

```c
HAL_UART_Receive(&huart1, (uint8_t *)rxBuffer, RX_BUFFER_SIZE, 5000);
```

* Waits for up to `5000 ms` for input.
* Stores up to `RX_BUFFER_SIZE` characters.

### Parsing commands

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

* Compares the received string with predefined commands.
* Updates LED state accordingly and sends feedback.

---

## How to Run

1. Connect STM32 board and ESP/Arduino (as USB-to-Serial converter) to PC.
2. Open **RealTerm** (or another serial terminal).
3. Set baud rate to **115200**, no parity, 1 stop bit.
4. Flash the code into STM32.
5. Observe the message prompt in the terminal.
6. Type `led0` or `led1` and check LED behavior on the STM32 board.
