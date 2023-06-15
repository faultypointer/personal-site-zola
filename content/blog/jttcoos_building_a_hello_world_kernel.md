+++
title = "Building A Hello World Kernel"
date = 2023-06-15
draft = false

[taxonomies]
categories = ["Journey to the center of operating system"]
tags = ["operating system", "systems programming", "assembly"]

[extra]
lang = "en"
toc = true
show_comment = true
math = false
mermaid = false
cc_license = true
outdate_warn = true
outdate_warn_days = 120
+++

## Intro
Reading about the workings and development of operating system is fascinating but there is still something missing. Getting your hands dirty with the code and racking your brain over something for hours only to give up and copying some code is way more interesting. Needless to say you shouldn't give up reading the theory — maybe you can? who am I to say. I am just getting started myself.

Reading theory and writing code in parallel is the way I like to do things .That is what I am gonna do. And obviously the best way to start is to write a hello world kernel. By write I mean copy it from [OSDevWiki](https://wiki.osdev.org/Main_Page).


## Before we get started
there are few things that we should know about so that atleast we understand what is going on rather than mindlessly following the guide.

### Cross compiler
Its a compiler that runs on a system(called Host) but generated the executable for another system(called Target. I am gonna show how I installed the GNU cross compiler for i686 system on linux so if you are on windows or mac get rekt. But also you can go to [osdevwiki](https://wiki.osdev.org/Building_GCC) to see how to build it on other system.
1. Installing the dependencies
```bash
	sudo dnf install gcc g++ make bison flex gmp-devel libmpc-devel mpfr-devel texinfo cloog-devel isl-devel
```
2. Download and extract the source code
	download the source code from [binutils](https://ftp.gnu.org/gnu/binutils/) and [gcc](https://ftp.gnu.org/gnu/gcc/) and put them where you like for eg ~/src.
```bash
	cd ~/src
	tar xf gcc.x.stuff binutils.x.stuff
```
3. Preparation
	decide where to install the binaries and for which system. ~/opt/ should be a good place to install. since i am compiling for i686, target system will be i686-elf
```bash
	export PREFIX="$HOME/opt/"
	export TARGET=i686-elf
```
4. Installing Binutils
```bash
cd ~/src/binutils.x.stuff
mkdir build && cd build
../configure --prefix="$PREFIX" --target="$TARGET" --disable-nls --disable-werror
make
make install
```
5. Installing GCC
```bash
cd ~/src/gcc.x.stuff
mkdir build && cd build
../configure --prefix="$PREFIX" --target="$TARGET" --disable-nls --enable-language=c,c++
make all-gcc
make all-target-libgcc 
make install-gcc
make install-target-libgcc
```

5. adding to path
	add this to the .bashrc
```bash
	export PATH="$HOME/opt/bin/:$PATH"
```

### Booting the system
- Existing piece of software is required to load our operating system which is called the bootloader. Popular bootloaders are GRUB, limine.
- Bootloader follows a certain protocol that outlines the communication and data exchange between the bootloader, firmware and the kernel.
- Multiboot is one such protocol. It enables the bootloader to pass important information line memory map, command line arguments and other details to the kernel.

### Freestanding and Hosted
A hosted binary is an executable that is compiled to run on a specific system. It relies on standrad library provided by the host system.

A freestanding binary is one that is compiled without a specific system in mind. As such it can't use the standrad library features.


## Writing the Kernel
Since we are writing a kernel that prints hello world on screen without using standrad library features, we will have to make use of vga text mode. Video Graphics Array(vga) text mode is a display mode that is supported by most modern computer. 

### Understanding VGA text mode
In VGA text mode, the screen is divided into a rectangular grid of fixed-size character cells. Each cell can display a single character from the ASCII character set and is associated with a specific foreground and background color.

Each screen character is represented by 16 bits accessible by the cpu. The 8 least significant bits are character bits that represent ascii character, 9th to 12th bits are for foreground color and rest of the bits are for background color. Depending on the mode setup, the most significant bit might be blink bit.

|Background      |Foreground      | Character           |
|----------------|----------------|---------------------|
|   0  0  0  0   |   0  0  0  0   |  0 0 0 0 0 0 0 0    |
 

#### VGA Color palette

|Color|integer|binary|
|-----|-------|------|
|black|0|0000|
|blue|1|0001|
|green|2|0010|
|cyan|3|0011|
|red|4|0100|
|magenta|5|0101|
|brown|6|0110|
|light grey|7|0111|
|dark grey|8|1000|
|light blue|9|1001|
|light green|10|1010|
|light cyan|11|1011|
|light red|12|1100|
|light magenta|13|1101|
|light brown|14|1110|
|white|15|1111|

so if you want to print white coloured 'H' on black background you would have to write this 16 bit sequence at the appropriate buffer index.
			0 0 0 0  |  1 1 1 1  |  0 1 0 0 1 0 0 0
		   Black bg  | white fg  | 'H' ascii 72

with this knowledge lets write a kernel that prints hello world on screen.

```cpp
#include <stdint.h> // stuff like size_t, uint8_t etc
#include <stddef.h>

// vga color palettes
enum class vga_color : uint8_t {
	VGA_COLOR_BLACK = 0,
	VGA_COLOR_BLUE = 1,
	VGA_COLOR_GREEN = 2,
	VGA_COLOR_CYAN = 3,
	VGA_COLOR_RED = 4,
	VGA_COLOR_MAGENTA = 5,
	VGA_COLOR_BROWN = 6,
	VGA_COLOR_LIGHT_GREY = 7,
	VGA_COLOR_DARK_GREY = 8,
	VGA_COLOR_LIGHT_BLUE = 9,
	VGA_COLOR_LIGHT_GREEN = 10,
	VGA_COLOR_LIGHT_CYAN = 11,
	VGA_COLOR_LIGHT_RED = 12,
	VGA_COLOR_LIGHT_MAGENTA = 13,
	VGA_COLOR_LIGHT_BROWN = 14,
	VGA_COLOR_WHITE = 15
};

// this function combines the foreground and background color
// to 8 color bits used by vga text mode.
static inline uint8_t vga_entry_color(vga_color fg, vga_color bg) {
	return static_cast<uint8_t>(fg) | static_cast<uint8_t>(bg) << 4;
}

// the 8 bit colors bits are then left shifted 8 bits to make room for charachter
// bits which is combined to be put in vga buffer.
static inline uint16_t vga_entry(unsigned char uc, uint8_t color) {
	return static_cast<uint16_t>(uc) | static_cast<uint16_t>(color) << 8;
}

// I don't think I need to explain this
size_t strlen(const char* str) {
	size_t len = 0;
	while(str[len]) {
	len++;
	}
	return len;
}

// set the size of vga rectangular grid to be used
static const size_t VGA_WIDTH = 80;
static const size_t VGA_HEIGHT = 25;

// global variables to be used for vga buffer and to keep
// track of current row, column and color
size_t terminal_row;
size_t terminal_column;
uint8_t terminal_color;
uint16_t* terminal_buffer;

// initialized the vga terminal
// the address 0xB8000 represents the starting address of the video memory buffer.
void terminal_initialize(void) {
	terminal_row = 0;
	terminal_column = 0;
	terminal_color = vga_entry_color(vga_color::VGA_COLOR_LIGHT_GREY, vga_color::VGA_COLOR_BLACK);
	terminal_buffer = reinterpret_cast<uint16_t*>(0xB8000);
	for (size_t y = 0; y < VGA_HEIGHT; y++) {
		for (size_t x = 0; x < VGA_WIDTH; x++) {
			const size_t index = y * VGA_WIDTH + x;
			terminal_buffer[index] = vga_entry(' ', terminal_color);
		}
	}
}

void terminal_setcolor(uint8_t color) {
	terminal_color = color;
}

void terminal_putentryat(char c, uint8_t color, size_t x, size_t y) {
	const size_t index = y * VGA_WIDTH + x;
	terminal_buffer[index] = vga_entry(c, color);
}

void terminal_putchar(char c) {
	// vga doesn't have support for newline character. so if the new line
	// character appears change to fisrt cell of next row
	if (c == '\n') {
		terminal_row++;
		terminal_column =0;
	return;
	}
	terminal_putentryat(c, terminal_color, terminal_column, terminal_row);
	// if the width is reached change the return the cursor to start of next row
	if (++terminal_column == VGA_WIDTH) {
		terminal_column = 0;
		// if the max row is reached go to the starting row
		if (++terminal_row == VGA_HEIGHT)
			terminal_row = 0;
	}
}

void terminal_write(const char* data, size_t size) {
	for (size_t i = 0; i < size; i++) {
		uint8_t clr = vga_entry_color(static_cast<vga_color>(i%15+1), vga_color::VGA_COLOR_BLACK);
		terminal_setcolor(clr);
		terminal_putchar(data[i]);
	}
}

void terminal_writestring(const char* data) {
	terminal_write(data, strlen(data));
}

extern "C" void kernel_main() {
	// Initialize terminal interface
	terminal_initialize();
	terminal_writestring("Hello, kernel World!\nYooooo New line support baby!!!");
}
```

### Compiling the code
The cross compiler we compiled will now be used to compile this code. To compile it:
```bash
i686-elf-g++ -c kernel.cpp -o kernel.o -ffreestanding -02 -wall -fno-exceptions -fno-rtti
```


## Assembly
We are now going to write an assembly code that serves as a entry point for the kernel by initializing some stuff.

### multiboot section
```asm
.set ALIGN, 1<<0
.set MEMINFO, 1<<1
.set FLAGS, ALIGN | MEMINFO
.set MAGIC, 0x1BADB002
.set CHECKSUM, -(MAGIC + FLAGS)

.section .multiboot
.align 4
.long MAGIC
.long FLAGS
.long CHECKSUM
```

we are defining some constants used in multiboot header and then defining multiboot section and populating with necessary values. The above code helps bootloader to confirm our kernel as multiboot compliant and validate headers integrity. This is done as per multiboot specification.

### bss section
```asm
.section .bss
.align 16
stack_bottom:
.skip 16384
stack_top:
```

the multiboot specification doesn't define the value of stack pointer register, so its upto ther kernel to provide a stack. the above code provides a 16KB stack from stack_bottom to stack_top. The align 16 is necessary because the stack on x86 must be 16-byte aligned accordind to System V ABI standrad.

### text section
```asm
.section .text
.global _start
.type _start, @function
_start:
    mov $stack_top, %esp
    call kernel_main
cli
1: hlt
jmp 1b
.size _start, . - _start

```

It defines the text section which contains executable instruction. We well be defining the \_start as the entry point in our linker script. \_start sets the stack pointer (%esp) to the top of the stack and then calls the main function of our c++ program. After kernel_main function returns, cli instruction disables the hardware interrupts and hlt and jmp instruction puts the cpu in infinite loop.

The last line sets the size of the \_start symbol to the current location '.' minus its start. This is useful when debugging or when you implement call tracing.

save the above code in boot.s file and the assemble it as:
```bash
i686-elf-as boot.s -o boot.o
```


## Linking the kernel
We now have seperate boot and kernel object files which we need to link to make a final kernel which can be used by bootloader. The default linker scripts is not suitable for this so we will write our own linker script.

```ld
/*defines the _start function as entry point into our kernel_*/
ENTRY(_start)

/*defines the sections and their location*/
SECTIONS {
	/*current memory location at 1 MB. conventional place for bootloader 
	to load kernel*/ 
    . = 1M;
	/*defines the code section of the program. should be aligned to a 4K boundary and that it should be split into blocks of 4K. multiboot section should be first otherwise bootloader won't recognize the file format.
    .text BLOCK(4K) : ALIGN(4K)
	{
		*(.multiboot)
		*(.text)
	}

    /* Read-only data. */
	.rodata BLOCK(4K) : ALIGN(4K)
	{
		*(.rodata)
	}
 
	/* Read-write data (initialized) */
	.data BLOCK(4K) : ALIGN(4K)
	{
		*(.data)
	}
 
	/* Read-write data (uninitialized) and stack */
    .bss BLOCK(4K) : ALIGN(4K)
	{
		*(COMMON)
		*(.bss)
	}
}
```

to link the kernel we do:
```bash
i686-elf-gcc -T linker.ld -o myos.bin -ffreestanding -02 -nostdlib boot.o kernel.o -lgcc
```

## Running the kernel
to run the kernel on qemu:
```bash
qemu-system-i386 -kernel myos.bin
```


<img src="/res/blogs/Screenshot_2023-06-15_13-42-21.png">