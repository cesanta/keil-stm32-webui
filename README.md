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

**Step 2. Start a new project.**
