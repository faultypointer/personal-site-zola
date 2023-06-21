+++
title = "On a quest to understand Basics of Operating System - OS Concepts"
date = 2023-06-21
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



In the first part of this blog series, we delved into the hardware components closely tied to operating systems. Now, let's shift our focus to the fundamental concepts that form the core of an operating system: processes, files, I/O, the shell, and system calls. Understanding these concepts will provide us with a deeper insight into how the operating system manages resources and enables efficient communication between the user and the hardware.

## Processes

A process is basically an instance of a program in execution. Each process has its **address space**, which is a list of memory locations that contains the executable program, program's data and its stack. The OS assigns each process a process ID (PID), manages its memory allocation, and handles its scheduling to use CPU time. The OS also handles process creation, termination, creation of child processes, and inter-process communication. The OS maintains info about each process's state (running, waiting, etc.), priority, resources used, and user ID.

## Files

Files serve as a means of storing and organizing data on secondary storage devices, such as hard drives. The operating system provides a file system that manages files, allowing users and programs to create, read, write, and delete files. A file system organizes data into directories or folders, providing a hierarchical structure for easy navigation and management. The operating system also ensures file security by implementing access control mechanisms, allowing only authorized users to access or modify files.

## I/O Management

Input/Output (I/O) management is a crucial aspect of any operating system. It enables the transfer of data between the computer and its peripheral devices, such as keyboards, mice, displays, and printers. The operating system abstracts the complexities of different devices and provides a unified interface for I/O operations. Device drivers, mentioned earlier, play a vital role in facilitating communication between the operating system and the devices, handling data transfers, and managing interrupts.

## The Shell

The shell acts as a command interpreter, serving as the interface between the user and the operating system. It provides a command-line or graphical interface through which users can interact with the system. The shell allows users to execute commands, run programs, navigate the file system, and manipulate files. Additionally, it supports features like scripting, batch processing, and job control, providing flexibility and automation capabilities to users.

## System Calls

System calls are the primary means by which user programs request services from the operating system. They act as an interface between user space and kernel space, allowing user programs to access privileged operations and interact with hardware resources. System calls provide functionalities such as process creation, file management, network communication, and device access. Examples of system calls include opening or closing a file, reading from or writing to a file, creating a new process, and allocating memory.

## Conclusion

Operating systems are complex entities that manage hardware resources and provide essential services to users and applications. By exploring concepts like processes, files, I/O management, the shell, and system calls, we gain a deeper understanding of how the operating system orchestrates these components to deliver a seamless computing experience.
