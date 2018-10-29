# Barebones multiboot kernel

- Intended for beginner OS development
- Install qemu for qemu-system-x86_64, gcc-multilib for a 32-bit compiler and nasm for the assembly
- Build and boot kernel in qemu with ./run.sh
- If building with -march=native (or anything that requires AVX), remember to run with KVM enabled (./run.sh --kvm) so that QEMU will present the modern instruction set features to the guest operating system.
- VGA textmode using ./run.sh --vga (NOTE: you must implement VGA support yourself! You will (if you're lucky) see a black screen.)
- Use ./build_iso.sh to run on real hardware or a hypervisor like VirtualBox, but keep serial port logging in mind!

## Features

- 600kb stack and working GDT
- Multiboot parameters passed to kernel start
- assert(), kprintf() and snprintf()
- Very basic heap implementation (malloc/free, new/delete)
- C and C++ global constructors
- Stack protector support
- EASTL C++ support, which has many useful containers
- Entire machine image is 32kb (8k without EASTL)
- ThinLTO support if you are using clang
    - Machine image goes from 32kb to 12kb with LTO

## Goal

The goal is to provide a barebones kernel project that implements the most basic C/C++ voodoo to let people start reading/writing to registers and devices immediately. The goal is also to provide people with a choice of using C or C++, or a mix of both.

## Future work

- Create simple page tables and directly enter long mode (64-bit)
    - If you create an issue about this I will drop in 64-bit long mode support. 32-bit is not what all the modern desktops out there are using right now!
- Add support for 64-bit sizes in the printf library

## Validate output

./run.sh output should match:
```
------------------
* Multiboot EAX: 0x2badb002
* Multiboot EBX: 0x9500
* SSE instructions ... work!
* Global constructors ... work!

Hello OSdev world!
This is kernel_main(uint32_t, uint32_t).

Press Ctrl+A -> X to close
Returned from kernel_start! Halting...
```

## Qemu defaults

Qemu provides you with some default devices to start with
- IDE for disk devices, but you can use PCI to get information
- e1000-compatible network card
- If you add -vga std you will get VGA graphics area at 0xb8000
- In graphics mode: PS/2 keyboard and mouse

As an example, use this function to clear the VGA textmode screen:
```
uint16_t* vga = (uint16_t*) 0xb8000;
for (int i = 0; i < 25*80; i++)
{
  vga[i] = ' ' | (7 << 8);
}
```
What it's actually doing is filling the text buffer with spaces. Spaces that have a gray color. You just can't see them, because spaces are spaces.
