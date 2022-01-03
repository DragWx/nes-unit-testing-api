# NES Unit Testing API
The following API aims to provide enough functionality to facilitate unit testing of 6502 code running on the NES or Famicom.

The goal of the API's structure is to provide a library which can be imported into your favorite development environment / build pipeline / programming language / etc, and within your favorite testing framework, you use this API to control the virtual NES.

The most basic and static features of the NES are considered first, such as built-in memory, common peripherals, and the most common cart-supplied memory.

At this time, there is no integration with individual mappers. Instead, functions to perform reads/writes to the NES's address space are provided as a way to interface with the current mapper.

## Setup Operations
* `loadRom(image)`: Loads the iNES-formatted ROM image into the virtual NES.
* `coldReset()`: Initiate a cold-reset of the virtual NES, like disconnecting power, waiting, then reconnecting power.
* `warmReset()`: Initiate a warm-reset of the virtual NES, like pressing the reset button.

For `reset` operations, battery-backed memory contents are retained only during the scope of the test, and not written to disk.

## Memory Operations
### In-Universe I/O
These functions will perform a memory operation just like the CPU would. Memory addressing depends on mappers, and any side-effects of accessing registers (e.g., mapper operations, address increments, etc) are enacted.
* `read(address)`
* `write(address)`

### Meta I/O
To access memory directly, several arrays are exposed:
* `ppuMem[]`: Internal PPU RAM
* `ppuCartMem[]`: External (cart-supplied) PPU RAM
* `ppuPaletteMem[]`: Internal PPU palette registers
* `ppuSpriteMem[]`: Internal sprite table RAM
* `cpuMem[]`: Internal CPU RAM
* `cpuCartMem[]`: External (cart-supplied) CPU RAM

Each array represents, if applicable, the memory *as it would physically appear on the memory module itself* (i.e., irrespective of banking).

## Code Execution
* `setCpuRegister(register, value)`: Set `A`, `X`, `Y`, `S`, `PC`, and flags.
* `runToReturn()`: CPU executes until a `return` underflows the stack.
* `runToBreakpoint(breakpoint)`: CPU executes until the `breakpoint` conditions are satisfied.

NOTE: There's no definition for `breakpoint` right now, but the idea is for it to be similar to one you'd supply to a debugger: read/write/execute address, optionally extra conditions like comparing memory addresses, cpu registers, etc.

## Performance
A user-resettable cumulative counter is available for custom metrics:
* Master clock cycle count
   * CPU cycle count, PPU pixel / scanline counts, etc, can be derived from this.


# Roadmap
Listed in no particular order.
* Direct access of mapper registers and memory
* Call/return-sensitive performance counters (shows performance for each subroutine called, sequentially and/or cumulatively)
* Memory hotspot map (shows how frequently each memory address is accessed)
* Component/Master-clock alignment


# Notes
## Symbols
This API only cares about physical addresses, but it would be nice if the user could specify, e.g., one of the labels from the assembly code and use *that* as the address. This would depend on ancillary assembler output, and there's multiple assemblers available.

* Use `od65` to parse object files from the `cc65` toolchain. (`--dump-exports` maybe?)
* Use the `-s` flag in `DASM` to generate a file with a list of all symbols.

Interpretation and resolution of these symbols is *not* in the scope of this API, and is better left to an external mechanism, such as another library, or something the user themself handles. However, here's my suggestion:
* Figure out how to get your assembler to output a list of `symbols` (i.e., labels and which values/addresses they resolve to) at assemble-time.
* Parse these symbols into something you can access at test-time (e.g., from a file into a lookup table addressed by label name).
* When you need to supply an address, get it from the data structure created from the previous step.