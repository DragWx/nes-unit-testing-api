This repository hosts the definition of an API which can be used to write unit
tests for the NES or Famicom.

The idea is:

* Choose a familiar and convenient programming environment to
write your tests in (e.g., C++, Python, Node.JS, etc).
* Choose a testing framework you're comfortable with.
* Import some kind of NES emulation library implementing this API.
* Write your unit tests with it.
   * Supply the emulation library with the iNES ROM image from your assembler's
     output.
   * Initialize the memory and various registers with any contents necessary for
     your tests.
   * Point the virtual CPU at a memory address and execute until return or some
     other kind of criteria, repeating if necessary.
   * Examine memory, registers, etc, and determine correctness.
