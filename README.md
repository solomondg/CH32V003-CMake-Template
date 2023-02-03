# CH32V003 CMake Template
### Ludicrously cheap microcontrollers, now in CLion or VSCode!

This is a basic template for developing on the WCH [CH32V003](http://www.wch-ic.com/products/CH32V003.html), including the vendor libraries and linker+startup files. Just clone/download the template, import to your IDE, and get started!

Note that you will need a RISC-V toolchain, I'm using the [xPack](https://github.com/xpack-dev-tools/riscv-none-elf-gcc-xpack) one. And, to program, you'll need a WCH-LinkE - note that the normal WCH-Link won't do the job. As well, you'll still need WCH's patched OpenOCD - sources [here](https://github.com/kprasadvnsi/riscv-openocd-wch), and a binary is included in WCH's MounRiver IDE (though, if you're here, you're probably trying to avoid that, like me).

## Project Structure
- **CMakeLists.txt** | Where the magic happens (and where I wrestled with the linker for half an hour)
- **Core** | Bare-bones RISC-V typedef and register/NVIC code. Sort of like CMSIS, if you're an ARM person.
- **Debug** | Contains some basic debug functions - delay, as well as the supporting functions for printf over USART1 (note - this is enabled by default - you'll have to modify it if you don't want that)
- **Ld** | Contains linker file. Modify this if you need to increase the amount of space reserved for the stack (512B default)
- **Peripheral** | WCH's vendor peripheral files. Feels very STM HAL-y.
- **Startup** | Contains system startup code - loading values into memory, and jumping to the main function
- **User** | Your code here! Contains some config files, syscall stubs, a file for interrupts, and your main.c! As well as any other code you write, of course ;) 

Toss an issue here if there's any questions or, of course, issues. 

### Happy hacking!
