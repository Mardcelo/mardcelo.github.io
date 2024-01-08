---
description: UEFI Definitions before UEFI development
---

# UEFI/BIOS/PI Definitions

## What is UEFI?

&#x20;Unified Extensible Firmware Interface is a spec that defines the architecture of the platform firmware used for booting the computer hardware and its interface for interaction with the operating system.

## Legacy BIOS and UEFI

* Legacy "system" means an outdated computer technology, application, or hardware that is still in use.
* Basic input/output system (BIOS) is the dominant standard that defines a firmware interface. "Legacy", in the context of firmware specifications, refers to an older, widely used specification
* The main responsibility of BIOS is to set hardware/load/start OS.
* When the computer boots, the BIOS initializes and identifies system devices including the video display card, keyboard, mouse, hard disk drive, and other hardware.
* The BIOS also locates software held on boot devices, and loads/executes that software giving it control of the computer. This is called "booting" or "bootstrapping".

## UEFI

* UEFI was a replacement for Legacy BIOS to streamline the booting process. And act as the interface between a computer OS and its platform firmware. It also offers a rich extensible pre-OS environment with advanced boot and runtime services.
* Unified Extensible Firmware Interface (UEFI) is grounded in Intel's initial Extensible Firmware. Interface (EFI) spec.
* UEFI architecture allows users to execute applications on a command line interface.
* It has intrinsic networking capabilities and is designed to work with multiprocessor (MP) systems.
* During the boot process, UEFI "Speaks" to the operating system loader and acts as the interface between the operating system and the BIOS.



## Intel Platform Innovation Framework&#x20;

### The Past&#x20;

* It was a framework
* It was implemented in C
* It was a set of robust architectural interfaces to accelerate the evolution of innovative, differentiated platform designs.
* The Framework was also the Intel-recommended implementation of the UEFI specification.

### The Present&#x20;

* Framework has been replaced with the more comprehensive UEFI Platform Initialization (PI) spec that complements the UEFI spec.
* UEFI and PI have differences
* UEFI is limited to programmatic interfaces for interactions between the OS and system firmware
* PI spec is a firmware implementation designed to perform the full range of operations required to initialize the platform from power-on through the transfer of control to the OS
* PI spec defines how a system gets from power-on to a state where UEFI is viable.
* PI also defines other hardware platform-spec elements needed by OS not incorporated in the UEFI spec.
* UEFI speaks to OS, UEFI does not address memory initialization, recovery, or platform initialization
* PI spec works within the infrastructure; PI takes the components you've built and defines how they interact properly to implement the boot process.

<div align="left">

<figure><img src="../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

</div>

* PI defines distinct phases for the boot process. PI defines services and constraints for the modules designed to run in that phase.
* Each phase builds on the previous until the system is ready for the OS.
* UEFI takes over to support the Option ROMs and the OS.
* Option ROM: Firmware that is called by the system BIOS. Option ROMs include BIOS firmware on add-on cards, as well as modules that extend the capabilities of the system BIOS.
