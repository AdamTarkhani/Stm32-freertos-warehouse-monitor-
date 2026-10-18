# STM32 FreeRTOS Warehouse Monitor

Built with the help of IEEE uOttawa.

A small FreeRTOS project for the STM32 Nucleo-F446RE board. It shows how several tasks share one microcontroller
and talk to each other without stepping on each other: a button stands in for a vibration sensor in an imaginary
warehouse, and the board reports every alert and a regular health message over the serial port.

## What it does

Three tasks run at the same time:

| Task | Priority | Job |
|---|---|---|
| `SensorTask` | High | Sleeps until the blue button is pressed. Then it flashes the green LED and sends the time of the event to the print task. |
| `PrintTask` | Normal | Waits for event times from the queue and prints `[ALERT] Vibration Sensor Triggered at <ms> ms!` over UART2. |
| `BlinkTask` | Low | Every 5 seconds prints a `[SYSTEM] Warehouse Monitor OK` health message. |

How the pieces connect:

- **Binary semaphore:** the button interrupt releases it, which wakes `SensorTask`. The task uses no CPU while it waits.
- **Message queue:** `SensorTask` puts the timestamp in the queue instead of printing, so the slow UART never delays the sensor task.
- **Mutex:** `PrintTask` and `BlinkTask` both print to the same UART, so each takes the mutex first and their messages never mix.
- **`printf` over UART2:** `printf` is redirected to the serial port, so output shows up in any serial terminal.

## Hardware

- STM32 Nucleo-F446RE (STM32F446RE)
- Blue user button (PC13) as the "sensor", green LED (PA5) as the alarm
- UART2 (PA2/PA3), which the board's USB connection exposes as a serial port

## Running it

1. Open STM32CubeIDE, choose File, Open Projects from File System, and pick this folder.
2. Open `Project.ioc` and click Generate Code. This recreates the ST HAL and FreeRTOS source folders, which are not in this repo.
3. Build and flash the board.
4. Open a serial terminal at the board's COM port and press the blue button.

## Layout

| Path | What it is |
|---|---|
| `Core/Src/main.c` | The tasks, the button interrupt callback and `printf` redirection |
| `Core/Src/freertos.c`, `Core/Inc/FreeRTOSConfig.h` | FreeRTOS setup |
| `Project.ioc` | The CubeMX configuration: pins, UART, clock, tasks, queue, semaphore and mutex |
| `STM32F446RETX_*.ld` | Linker scripts |
