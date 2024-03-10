# Building Web UI Dashboard on STM32 Nucleo-F756ZG

This guide is a step-by-step tutorial on how to build Web UI dashboard
on STM32 Nucleo-F756ZG development board using Keil IDE. Prerequisites:
- Hardware: Nucleo-F756ZG, connected to the workstation and to Ethernet
- Software: Keil uVision

## General plan

The first step is to create a minimal firmware that:
- initialises system clock to the maximum frequency
- intialises debug UART and redirects `printf()` to the debug UART
- outputs some debug message periodically

After that minimal firmware is created, we'll add a minimal web server on top
of it, and then enhance that minimal web server to a fully functional Web
Device Dashboard.

## Minimal app

**Step 1. Install necessary packs.** Start Keil, click on "Pack Installer"
to start an installer. There in the search input, type "stm32f7", then click
on "STM32F7 Series". On the right side of the window, you should see
"Device Specific / Keil::STM32F7xx_DFP" pack, click on "Install" if it is not
yet installed. When installed, install "Generic / Cesanta::Mongoose" pack  

**Step 2. Start a new project.** Create a new project. In the Search filter,
type "stm32f756zg", select first device. In the RTE window, check
"Device/Startup", click on "Resolve", click OK.

**Step 3. Create main.c.** Right click on "Source group 1", "Add new file",
"C file", type "main", click "Add". Copy/paste the following content, then save:
```c
int main(void) {
  return 0;
}
```

**Step 4. Update settings.** Click on "Options for target" in the toolbar,
- On the "Device Target", check "Use multilib"
- On the "User" tab, uncheck "Beep when complete"
- On the C/C++ tab, select Warnings / MISRA compatible
- On the Debug tab, select Use / ST-Link debugger
- Click on debugger "Settings", "Flash Download" tab, check "Reset and Run"

**Step 5. Build project.** Click on "Built" toolbar button. The project should
be built with zero errors and warnings.

**Step 6. Configure the board.** Instead of using CubeMX to configure clock,
UART and LED, we'll copy a small HAL header and use it to achieve the same
in a much quicker way.
- Open [hal.h](https://mongoose.ws/demo/?board=Nucleo-F756ZG&project=mqtt&file=hal.h)
copy file contents to the clipboard.
- Right click on "Source Group1", choose "Add new file", name it "hal.h",
  paste the contents, save
- Do the same with hal.c - copy it into the project
- Update main.c code with the following:

```c
#include "hal.h"

extern uint64_t mg_millis(void);

int main(void) {
  hal_init();

  uint64_t timer = 0;
  for (;;) {
    if (timer < mg_millis()) {
      timer += 500;
      gpio_toggle(LED1);
      printf("hi! tick is %lu\r\n", (unsigned long) mg_millis());
    }
  }

return 0;
}
```
- Click on "Load" toolbar button to flash this firmware
- The green LED should start blinking
- Open device manager, look up what COM port device corresponds to your board
- Start putty, and open a serial port with baud 115200
- You should see messages like this:
  ```
  hi! tick is 1559001
  hi! tick is 1559501
  hi! tick is 1560001
  ```
- Congratulations! We have minimal skeleton firmware working

## Integrate Mongoose Library


**Step 1. Enable Mongoose Library.** Click on "Manage Run-Time Environment",
and enable "Network Library" / "Mongoose"  

**Step 2. Configure Mongoose Library.** In the project view, click on
"Network Library" / `mongoose_custom.h`. Click on the "Configuration Wizard"
tab and make sure the following options are set:
- Build environment - Keil MDK
- Built-in TCP/IP stack - enabled
- Enable STM32Fxx driver - enabled
- TLS support - none
- Enable custom `mg_millis()` - enabled
- Enable custom `mg_random()` - enabled
- Enable packed filesystem - disabled

**Step 3. Add mongoose header.** Open `main.c`, add `#include "mongoose.h"` at
the top, remove `mg_millis()` declaration. Click on build to make sure the
project builds.
