# Custom Pipelined RV32IM Logisim Diagram

This is my custom pipelined implementation of the RV32IM instruction set in [Logisim Evolution v3.90dev](https://github.com/logisim-evolution/logisim-evolution). It can run compiled code from the [riscv-gnu-toolchain](https://github.com/riscv-collab/riscv-gnu-toolchain) as well as draw to the screen. Currently, [my custom library](https://github.com/ryanriccio1/RV32IMLib) is not set up to support this, since graphics and memory optimizations have been made since the initial logic design. On top of this, due to limitations of Logisim, the system clock is limited by the VGA clock since we cannot have dual-read memory. Some shortcuts were taken to allow compatibility with these limitations that were not implemented in my [emulator](https://github.com/ryanriccio1/RV32IMEmulator). As this was for a capstone project, my [final paper](final_paper.pdf) has been attached for those seeking more information into the overall design of this architecture, however the general description is as follows.

Overall, the project is broken into libraries that come together to form the computer.

1. [computer.circ](#computercirc)
1. [riscv_core.circ](#riscv_corecirc)
1. [ram.circ](#ramcirc)
1. [vga.circ](#vgacirc)
1. [peripheral_interface.circ](#peripheral_interfacecirc)
1. [memory_mgmt.circ](#memory_mgmtcirc)

## computer.circ

![computer.circ](images/computer.png)

This is the main library that combines the functional components into a computer. Similar to a Commodore 64, it has a peripheral controller as well as a MMU which basically acts as a chip select for a given memory address. It has a keyboard input, TTY output, a reset button, 64x64 VGA screen, a button that triggers an interrupt, and the main clock source.

## riscv_core.circ

![riscv_core.circ](images/core.png)

Modeled initially using a standard RISC style design, similar to that of MIPS, this is the main implementation of the pipelined RV32IM core. It's program memory and data memory are unified and external to the core, unlike traditional MIPS/RISC style designs. This aligns with the Commodore 64's ability to memory map anything and everything, regardless of bounds. While this might be unsafe, it is a tinkerer's dream for a custom, proof of concept architecture like this.

## ram.circ

![ram.circ](images/ram.png)

In my RAM library, I have single, double, and quadruple read RAM circuits. This works on the fundamental basis that the clock cycle of the RAM is twice that of the rest of the computer. This is done with a RAM CLK circuit that uses a counter in a similar fashion as a flip flop to cut the clock cycle frequency in half for the rest of the computer. On the first half of the clock cycle, the RAM is written to, and on the second half, it is read from. Each bank of memory is identical to each other, however they can be read at different addresses.

## vga.circ

![vga.circ](images/vga.png)

The VGA circuit is modeled similar to how it would be implemented in real life. The counter is obviously modified to work with the Logisim timings/screen constraints, however the control registers and output calculation are very similar. The VGA controller has two modes, an 8x8 character mode and a 24bit RGB bitmap mode. The character mode consists of a color ROM (foreground/background for each 8x8 cell), a character ROM (indexed array of 8x8 characters), and the same VGA memory is used as an 1-byte index of which character is at a given position (i.e. a 2D array of characters on the screen). The output calculation is a bit of a hodgepodge of bit arithmetic, however at the end of the day, it takes in memory vectors and screen positions, and outputs lookup addresses and RGB values.

The control registers are also memory mapped and allow for all of the memory used by the VGA card to be dynamically addressed/allocated. In my example programs, the VGA memory addresses get allocated at runtime and placed on the heap, although these control registers are not implementation specific. In the final emulator, the control registers were dynamically allocated, allowing for memory to expand and contract, however due to the limitations of Logisim, this was not emulated in these logic drawings.

## peripheral_interface.circ

![peripheral_interface.circ](images/pia.png)

While the PIA is loosely based off the [Veecom VIA](https://github.com/MazinCE/Veecom?tab=readme-ov-file#versatile-interface-adapter-via-), it has been optimized for IRQ support and UART control. It functions as a gateway to allow peripherals to be memory mapped.

## memory_mgmt.circ

![mmu](images/mmu.png)

The MMU is a glorified look up table that converts the current address into a "chip select" style signal for each of the functional units of the computer.

## Fun VGA Demo

![my face](images/my_face.png)

Here is a C program I wrote as a VGA test that uses bitmap mode to draw a 64x64 image of my profile picture.
