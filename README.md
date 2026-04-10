[![Review Assignment Due Date](https://classroom.github.com/assets/deadline-readme-button-22041afd0340ce965d47ae6ef1cefeee28c7c493a6346c4f15d667ab976d596c.svg)](https://classroom.github.com/a/57vxu_4B)

---
Dana Khalil: g00098584
Omar Matar: b00099010
Farah Tawalbeh: g00099768
---

# COE411L-00 Report 3
### Introduction
This lab introduces the use of the ESP32 microcontroller within the PlatformIO environment using the ESP-IDF framework. Unlike Arduino-based development, ESP-IDF provides a lower-level interface with direct access to hardware and built-in FreeRTOS support, making it suitable for real embedded systems.

The main objective of this lab is to understand how to configure and run ESP32 projects, simulate them using Wokwi, and implement basic embedded functionalities such as logging, digital input/output, and PWM-based control. Additionally, the lab reinforces the use of structured debugging through serial logging.

### Description

In this lab, three main systems were developed:

## 1. Logging System (Experiment 1)
A simple ESP32 application was implemented to continuously print log messages using the ESP-IDF logging system. The program runs inside the `app_main()` function, which acts as the entry point and executes as a FreeRTOS task. Messages are printed periodically using `ESP_LOGI()` along with a delay function to control execution timing.

## 2. Digital Input/Output System (Experiment 2)
A system was built to control an LED using a pushbutton. The GPIO pins were configured using ESP-IDF drivers:
- The LED pin was configured as an output.
- The pushbutton pin was configured as an input with an internal pull-up resistor.

The system continuously reads the button state and updates the LED accordingly:
- Button pressed: LED ON  
- Button released: LED OFF  

This demonstrates real-time hardware interaction using GPIO APIs.

### 3. Servo Control System using PWM (Experiment 3)
A servo motor was controlled using the ESP32’s LEDC (LED Controller) module, which generates PWM signals. The system:
- Configures a PWM timer at 50 Hz (required for servo control)
- Uses a PWM channel to output the signal to the servo pin
- Adjusts the duty cycle to control the servo angle (0°, 90°, 180°)

A helper function converts pulse width (in microseconds) to duty cycle values. The servo continuously sweeps between angles in an infinite loop.

### Key Code Segments
## Exercise 1

1. Library Inclusions and Tag Definition
```
#include <stdio.h>
#include "freertos/FreeRTOS.h"
#include "freertos/task.h"
#include "esp_log.h"

static const char *TAG = "LAB1";
```
This section includes the required libraries for:

standard input/output (stdio.h),
FreeRTOS functionality (task handling and delays),
and the ESP-IDF logging system (esp_log.h).

The TAG variable is used to label log messages, making it easier to identify their source in the serial monitor.

2. Main Application Function
```
void app_main(void)
{
    ...
}
```
app_main() is the entry point of the program in ESP-IDF. Unlike Arduino, there is no setup() or loop() function. Instead, all execution starts here, and it runs as a FreeRTOS task.

3. Logging Output
```
ESP_LOGI(TAG, "Hello from ESP-IDF!");
```
This line prints a message to the serial monitor using the ESP-IDF logging system: ESP_LOGI indicates an informational log level, TAG identifies the source of the message, the string is the actual message displayed.

This structured logging is more useful than simple printf for debugging embedded systems.

4. Task Delay
```
vTaskDelay(pdMS_TO_TICKS(1000));
```
This introduces a delay of 1000 milliseconds (1 second) using FreeRTOS: pdMS_TO_TICKS() converts milliseconds into RTOS ticks, vTaskDelay() pauses the task without blocking the system.

5. Continuous Execution Loop
```
while (1)
{
    ESP_LOGI(TAG, "Hello from ESP-IDF!");
    vTaskDelay(pdMS_TO_TICKS(1000));
}
```
The infinite loop ensures that the log message is printed repeatedly every second. This demonstrates periodic task execution using FreeRTOS.

## Exercise 2

1. Pin Definitions
```
#define LED_PIN GPIO_NUM_2
#define BUTTON_PIN GPIO_NUM_19
```
These macros define the GPIO pins used in the system:
LED_PIN: connected to the LED (output)
BUTTON_PIN:connected to the pushbutton (input)

2. GPIO Configuration
```
gpio_reset_pin(LED_PIN);
gpio_set_direction(LED_PIN, GPIO_MODE_OUTPUT);

gpio_reset_pin(BUTTON_PIN);
gpio_set_direction(BUTTON_PIN, GPIO_MODE_INPUT);
gpio_pullup_en(BUTTON_PIN);
```
This section initializes and configures the GPIO pins:

gpio_reset_pin() ensures the pins start from a clean default state.
The LED pin is set as an output so it can be controlled by the program.
The button pin is set as an input to read user interaction.
gpio_pullup_en() enables the internal pull-up resistor, meaning:
Default state: HIGH (1)
Pressed state: LOW (0)

3. Reading Button Input
```
int state = gpio_get_level(BUTTON_PIN);
```
This reads the current logic level of the pushbutton:

1: button not pressed
0: button pressed

4. LED Control Logic
```
if (state == 0)
{
    gpio_set_level(LED_PIN, 1);
}
else
{
    gpio_set_level(LED_PIN, 0);
}
```
This implements the system behavior:

When the button is pressed (state == 0), the LED is turned ON
When the button is released (state == 1), the LED is turned OFF

5. Continuous Execution Loop
```
while (1)
{
    ...
}
```
The infinite loop ensures the system continuously: Reads the button state and updates the LED accordingly.

This creates a real-time response system where the LED instantly reflects user input.

## Exercise 3

1. Library Inclusions and Definitions
```
#include "freertos/FreeRTOS.h"
#include "freertos/task.h"
#include "driver/ledc.h"

#define LEDC_TIMER       LEDC_TIMER_0
#define LEDC_MODE        LEDC_LOW_SPEED_MODE
#define SERVO_GPIO       GPIO_NUM_18
#define LEDC_CHANNEL     LEDC_CHANNEL_0
#define LEDC_DUTY_RES    LEDC_TIMER_8_BIT
#define SERVO_FREQ_HZ    50
```
This section includes the required libraries for FreeRTOS delays and PWM control using the LEDC driver. It also defines the main configuration values used in the program:

SERVO_GPIO specifies the pin connected to the servo signal wire.
SERVO_FREQ_HZ is set to 50, which is the standard PWM frequency for servo motors.
LEDC_TIMER, LEDC_CHANNEL, and LEDC_MODE identify the PWM timer and channel being used.
LEDC_DUTY_RES sets the PWM resolution to 8 bits.

2. PWM Timer Configuration
```
ledc_timer_config_t ledc_timer = {
    .speed_mode = LEDC_MODE,
    .timer_num = LEDC_TIMER,
    .duty_resolution = LEDC_DUTY_RES,
    .freq_hz = SERVO_FREQ_HZ,
    .clk_cfg = LEDC_AUTO_CLK
};
ledc_timer_config(&ledc_timer);
```
This code configures the PWM timer. The timer determines the frequency and resolution of the generated PWM signal. In this case: the frequency is fixed at 50 Hz for servo operation, the duty resolution is 8-bit, and the timer is automatically assigned a clock source. This step is necessary before generating any PWM output.

3. PWM Channel Configuration
```
ledc_channel_config_t ledc_channel = {
    .gpio_num = SERVO_GPIO,
    .speed_mode = LEDC_MODE,
    .channel = LEDC_CHANNEL,
    .intr_type = LEDC_INTR_DISABLE,
    .timer_sel = LEDC_TIMER,
    .duty = 0,
    .hpoint = 0
};
ledc_channel_config(&ledc_channel);
```
This section attaches the configured PWM signal to the servo pin. It selects the GPIO pin connected to the servo, the PWM channel that will output the signal, and the timer that controls that channel. The initial duty cycle is set to 0, so the servo starts in an inactive state until updated in the loop.

4. Servo Position Control
```
ledc_set_duty(LEDC_MODE, LEDC_CHANNEL, 7);
ledc_update_duty(LEDC_MODE, LEDC_CHANNEL);
vTaskDelay(pdMS_TO_TICKS(1000));
```
This pattern is repeated to move the servo to different positions. The code sets a duty value, updates the PWM output, and waits for one second before changing to the next position.

The duty values used are:
7 for approximately 0°
19 for approximately 90°
31 for approximately 180°

These values were chosen to produce suitable pulse widths for the servo in simulation.

5. Continuous Sweep Motion
```
while (1)
{
    ledc_set_duty(LEDC_MODE, LEDC_CHANNEL, 7);
    ledc_update_duty(LEDC_MODE, LEDC_CHANNEL);
    vTaskDelay(pdMS_TO_TICKS(1000));

    ledc_set_duty(LEDC_MODE, LEDC_CHANNEL, 19);
    ledc_update_duty(LEDC_MODE, LEDC_CHANNEL);
    vTaskDelay(pdMS_TO_TICKS(1000));

    ledc_set_duty(LEDC_MODE, LEDC_CHANNEL, 31);
    ledc_update_duty(LEDC_MODE, LEDC_CHANNEL);
    vTaskDelay(pdMS_TO_TICKS(1000));

    ledc_set_duty(LEDC_MODE, LEDC_CHANNEL, 7);
    ledc_update_duty(LEDC_MODE, LEDC_CHANNEL);
    vTaskDelay(pdMS_TO_TICKS(1000));
}
```
The infinite loop makes the servo continuously move through a sequence of positions from 0° to 90° to 180° then back to 0°

A one-second delay is inserted between each movement so that the position change is clearly visible in simulation.
  
### Screenshots
## Exercise 1
<img width="800" height="360" alt="image" src="https://github.com/user-attachments/assets/c8c849a4-eaec-4080-a7fb-2c8a75dbcf72" />
<img width="1434" height="826" alt="image" src="https://github.com/user-attachments/assets/95cb15ba-4a72-4e83-a2f0-4987667abcff" />

## Exercise 2
Button is not pressed:
<img width="1017" height="831" alt="image" src="https://github.com/user-attachments/assets/d422d269-a9dc-4a8b-acd3-2738cad018c4" />

Button is pressed:
<img width="868" height="670" alt="image" src="https://github.com/user-attachments/assets/266abeb6-34f7-44c6-8dd4-924c4260cde1" />

## Exercise 3
0 degrees:
<img width="1009" height="795" alt="image" src="https://github.com/user-attachments/assets/11057ae8-a91b-4738-8ebe-0eae859a320b" />
90 degrees:
<img width="1008" height="861" alt="image" src="https://github.com/user-attachments/assets/6d18c4ee-fa9b-4b3f-9241-a2f79bc044a2" />
180 degrees:
<img width="998" height="832" alt="image" src="https://github.com/user-attachments/assets/94e4e13e-1887-458a-930a-13608d260243" />


### Conclusion

In this lab, we successfully explored the ESP32 development workflow using PlatformIO and the ESP-IDF framework. Through the three experiments, we gained practical experience with core embedded system concepts including structured logging, digital input/output control, and PWM-based actuator control.

The first experiment demonstrated the use of the ESP-IDF logging system for debugging and monitoring program execution. The second experiment reinforced fundamental GPIO operations by implementing real-time interaction between a pushbutton and an LED. The third experiment introduced PWM generation using the LEDC module, allowing precise control of a servo motor.

Overall, this lab provided a clear transition from higher-level embedded development to a more advanced, low-level framework with direct hardware access and FreeRTOS integration. The use of Wokwi simulation further enhanced development efficiency by enabling testing and debugging without physical hardware.
