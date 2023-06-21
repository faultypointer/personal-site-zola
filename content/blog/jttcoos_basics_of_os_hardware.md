+++
title = "On a quest to understand Basics of Operating System - The Hardware"
date = 2023-06-17
draft = false

[taxonomies]
categories = ["Journey to the center of operating system"]
tags = ["operating system", "computer hardware", "basics of os"]

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



Since operating system is closely tied to the hardware of the computer it operates upon, understanding the underlying hardware makes it a bit easy to understand the workings of operating system. So let us start reviewing the various components of computer hardware in brief


## Processors

Processors serve as the central processing unit (CPU) and act as the brain of the computer. They function in a continuous cycle, where they retrieve instructions from the memory, decode and execute them, and move on to the next instruction. Processors contain registers that store essential variables and temporary results. Additionally, they possess special registers that play distinct roles:

- Program Counter: Holds the memory address of the next instruction to be fetched.
- Stack Pointer: Indicates the top of the current stack in memory.
- Program Status Word: Contains condition code bits.

Modern computers employ a pipeline model, enabling the execution of multiple instructions simultaneously.

Another critical aspect of modern operating systems is the presence of two modes in which programs run: kernel mode and user mode. In kernel mode, the processor can utilize all hardware features and execute every instruction in the instruction set. On the other hand, user mode imposes restrictions on certain instructions and allows only a subset of instructions to be executed. Programs running in user mode must make a _system call_ to obtain services from the operating system.

## Memory: Storing and Accessing Data

The memory of a computer system is constructed in a hierarchical manner, with each layer possessing varying characteristics in terms of speed, capacity, and cost per bit.

### Registers: The Fastest Memory Layer

Registers represent the top layer of the computer memory hierarchy. Composed of the same material as the CPU, registers offer lightning-fast access. Typically, registers have a storage capacity of less than 1 KiB.

### Cache: Bridging the Gap

Cache serves as the second layer of the memory hierarchy. Modern CPUs have two levels of cache. L1 cache resides inside the CPU and feeds instructions to the CPU. L2 cache retains recently used memory words, ensuring swift access.

### RAM: Main Memory

The next layer in the memory hierarchy is the main memory or Random Access Memory (RAM). All data requests that cannot be fulfilled by the cache are directed to the main memory.


## I/O Devices

Operating systems extensively interact with input/output (I/O) devices. These devices generally consist of two components: a controller and the device itself. The controller is a chip that physically manages the device and provides a simplified interface to the operating system. The software responsible for communicating with the controller is known as a device driver.

## Buses

Buses serve as communication pathways, connecting different hardware components within the computer. They facilitate the exchange of data and instructions between the CPU, memory, and I/O devices. Several types of buses are found in a computer system:

### System Bus
The system bus, also referred to as the front-side bus or memory bus, facilitates the transfer of data between the CPU and the main memory. It carries memory addresses, data, and control signals. The width of the system bus determines the volume of data transferred in a single bus cycle.

### I/O Bus

The I/O bus, known as the peripheral bus or expansion bus, establishes the connection between the CPU and various I/O devices such as hard drives, graphics cards, network cards, and USB devices. Different types of I/O buses, such as PCI (Peripheral Component Interconnect) and USB (Universal Serial Bus), feature their own specifications and speeds.

### Control Bus

The control bus transmits control signals between the CPU and other hardware components. These signals include read and write signals to indicate data transfer operations, interrupt signals to request attention from the CPU, and clock signals to synchronize the operations of different components.

### Address Bus

The address bus carries memory addresses from the CPU to the main memory or I/O devices. It determines the location in memory where data needs to be read from or written to. The width of the address bus determines the maximum addressable memory.

### Data Bus

The data bus transfers actual data between the CPU, memory, and I/O devices. It conveys the data being read from or written to memory or I/O devices. The width of the data bus determines the number of bits that can be transferred in a single bus cycle.

## Conclusion

A solid understanding of computer hardware is crucial for comprehending the inner workings of an operating system. Processors, memory, I/O devices, and buses are integral components that collaborate with the operating system to execute instructions, store and retrieve data, and facilitate seamless communication between different hardware elements. By grasping these hardware concepts, we can gain a clearer understanding of how the operating system interacts with the underlying hardware to provide a smooth computing experience.