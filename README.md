````markdown
# OV7670 Real-Time FPGA Image Processing System

A fully hardware-accelerated real-time image processing pipeline implemented on FPGA using Verilog HDL. The system captures live video from an OV7670 camera module, performs selectable convolution-based image filtering in hardware, and outputs the processed video stream to a VGA display at 640×480 resolution.

---

# Project Overview

This project demonstrates an end-to-end real-time computer vision pipeline implemented entirely in FPGA fabric without the use of any embedded processor or software stack.

The OV7670 camera continuously streams pixel data into the FPGA, where each RGB channel is processed independently through a parallel 3×3 convolution engine. The processed frame is stored in on-chip Block RAM and simultaneously displayed through a VGA controller.

The architecture emphasizes:

- Fully parallel hardware image processing
- Real-time convolution filtering
- Multi-clock domain synchronization
- Efficient BRAM-based frame buffering
- Modular and scalable RTL design

---

# System Architecture

```text
               ┌────────────────────┐
               │    OV7670 Camera   │
               └─────────┬──────────┘
                         │
                         ▼
        ┌────────────────────────────────┐
        │            cam_top             │
        │  • SCCB Camera Initialization  │
        │  • Pixel Capture Logic         │
        └────────────────┬───────────────┘
                         │
                  RGB444 Pixel Stream
                         │
                         ▼
        ┌────────────────────────────────┐
        │        imageProcessTop         │
        │                                │
        │   ┌────────┐   ┌────────┐      │
        │   │ image  │   │ conv   │      │
        │   │Control │→→│ Engine │      │
        │   └────────┘   └────────┘      │
        │                                │
        │   Parallel R/G/B Processing    │
        └────────────────┬───────────────┘
                         │
                 Processed RGB444
                         │
                         ▼
        ┌────────────────────────────────┐
        │           mem_bram             │
        │     Frame Buffer Storage       │
        └────────────────┬───────────────┘
                         │
                         ▼
        ┌────────────────────────────────┐
        │            vga_top             │
        │   VGA Timing + Pixel Output    │
        └────────────────┬───────────────┘
                         │
                         ▼
                 VGA Display Output
````

---

# Key Features

* Real-time video capture from OV7670 camera
* Fully hardware-based image processing pipeline
* Parallel RGB channel convolution architecture
* Selectable convolution kernels in real time
* VGA output at 640×480 @ 60 Hz
* Dual-port BRAM frame buffering
* Multi-clock domain synchronization
* Pure Verilog HDL implementation
* No CPU, firmware, or external software processing

---

# RTL Module Description

| Module              | Functionality                                                                       |
| ------------------- | ----------------------------------------------------------------------------------- |
| `top.v`             | Top-level integration module handling clocking, resets, and subsystem interconnects |
| `cam_top.v`         | Camera interface subsystem including initialization and pixel acquisition           |
| `cam_init.v`        | Configures OV7670 registers over SCCB interface                                     |
| `cam_capture.v`     | Captures camera pixel stream and generates RGB444 pixels                            |
| `cam_config.v`      | Contains OV7670 configuration parameters                                            |
| `cam_rom.v`         | ROM storing SCCB register address/data sequences                                    |
| `sccb_master.v`     | SCCB communication controller for OV7670 configuration                              |
| `imageProcessTop.v` | Top-level image processing pipeline                                                 |
| `imageControl.v`    | Generates sliding 3×3 pixel windows using line buffering                            |
| `conv.v`            | Hardware convolution engine supporting multiple kernels                             |
| `lineBuffer.v`      | Line-buffer memory for neighborhood pixel generation                                |
| `mem_bram.v`        | Dual-port BRAM frame buffer                                                         |
| `vga_top.v`         | VGA subsystem integration module                                                    |
| `vga_driver.v`      | VGA timing generator and synchronization controller                                 |
| `debounce.v`        | Push-button debouncing logic                                                        |

---

# Real-Time Convolution Filters

The active filter is selected using the 3-bit `i_kernel_sel` input.

| Kernel Select | Filter                  |
| ------------- | ----------------------- |
| `000`         | Identity / Pass-through |
| `001`         | Sobel Edge Detection    |
| `010`         | Box Blur                |
| `011`         | Negative                |
| `100`         | Sharpen                 |
| `101`         | Emboss                  |
| `110`         | Edge Enhancement        |
| `111`         | Custom Kernel           |

---

# FPGA Resources and IP Cores

| IP Core        | Purpose                                         |
| -------------- | ----------------------------------------------- |
| `clk_wiz_1`    | Clock generation for VGA timing and OV7670 XCLK |
| `outputBuffer` | AXI4-Stream FIFO for clock-domain buffering     |

---

# Top-Level Interface

| Signal                 | Direction | Description                         |
| ---------------------- | --------- | ----------------------------------- |
| `i_top_clk`            | Input     | System clock (100 MHz)              |
| `i_top_rst`            | Input     | Active-low system reset             |
| `i_kernel_sel[2:0]`    | Input     | Real-time filter selection          |
| `i_top_cam_start`      | Input     | Camera initialization trigger       |
| `o_top_cam_done`       | Output    | Camera initialization complete flag |
| `i_top_pclk`           | Input     | OV7670 pixel clock                  |
| `i_top_pix_byte[7:0]`  | Input     | OV7670 pixel data bus               |
| `i_top_pix_vsync`      | Input     | Vertical synchronization input      |
| `i_top_pix_href`       | Input     | Horizontal reference input          |
| `o_top_siod`           | Output    | SCCB serial data                    |
| `o_top_sioc`           | Output    | SCCB serial clock                   |
| `o_top_xclk`           | Output    | External clock to OV7670            |
| `o_top_vga_red[3:0]`   | Output    | VGA red channel                     |
| `o_top_vga_green[3:0]` | Output    | VGA green channel                   |
| `o_top_vga_blue[3:0]`  | Output    | VGA blue channel                    |
| `o_top_vga_hsync`      | Output    | VGA horizontal sync                 |
| `o_top_vga_vsync`      | Output    | VGA vertical sync                   |

---

# Development Environment

## Hardware

* Xilinx 7-Series FPGA Board

  * Basys 3
  * Nexys A7
* OV7670 Camera Module (without FIFO)
* VGA-Compatible Monitor

## Software

* Xilinx Vivado 2020.x or later

---

# Build and Deployment

1. Clone the repository
2. Create a new Vivado project
3. Add all RTL files from the `rtl/` directory
4. Import IP core `.xci` files from `ip_files/`
5. Add the appropriate FPGA constraint (`.xdc`) file
6. Run synthesis and implementation
7. Generate and program the bitstream
8. Press the `cam_start` button to initialize the camera
9. Change `kernel_sel` switches to apply filters in real time

---

# Repository Structure

```text
├── rtl/                 # Verilog RTL source files
├── ip_files/            # Vivado IP core configuration files
├── constraints/         # FPGA constraint files (.xdc)
├── sim/                 # Optional simulation testbenches
└── README.md
```

---

# Applications

* FPGA-based computer vision systems
* Real-time embedded image processing
* Edge detection and feature extraction
* Hardware acceleration research
* Digital signal processing education
* Low-latency video processing systems

---

# Future Improvements

* HDMI/DVI video output support
* Higher-resolution image processing
* Additional convolution kernels
* Dynamic kernel coefficient loading
* Hardware histogram equalization
* Object detection preprocessing pipeline
* AXI-stream compatible architecture
* DDR-based external frame buffering

---

# License

This project is intended for educational and research purposes.
Please credit the original author when reusing or modifying the design.

```
```
