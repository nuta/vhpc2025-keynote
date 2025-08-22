# What I'm going to talk about

- First, let me clarify what I'm going to talk about.
- This talk is not really about writing a hypervisor from scratch in 40 minutes.
- This talk is also not about being able to write a hypervisor from scratch.

- Instead, this talk is to rediscover the hardware-assisted virtualization.
- It's not just for running virtual machines. It has a lot of potential.

# Hardware-assisted virtualization

- I know you are already familiar with this, so let me go through it quickly.
- Hardware-assisted virtualization is a CPU feature which introduces a new CPU mode, called "guest mode".
- Once CPU enters the guest mode, CPU executes instructions as usual, and when the guest needs an assistance from the hypervisor, the CPU exits the guest mode and jumps to the hypervisor (or host) kernel, which is called "vm exit".
- The hypervisor handles the vm exit, like emulating sensitive instructions, emulating memory-mapped I/O, and go back to the guest mode.

# Cover: Hardware-assisted try-catch (with continuation)

- TODO:

# Try-catch pattern

- So what's try-catch?
- I know you know it, but let's take a look from the beginning.
- Here's a simple example in pseudo code.
- In `try` block, the program does something that might fail.
- If it fails, the program jumps to the `catch` block.
- In other words, `catch` block is a handler to help the program. TODO:

# Try-catch pattern + continuation

TODO:

# Writing a hypervisor from scratch

TODO:

- Let's discover this try-catch pattern in a hypervisor.

# Target

- In this quick tutorial:
  - I'll use Rust programming language.
  - The target is 64-bit RISC-V CPU with hypervisor extension.
  - Emulated by QEMU.
- Again, this is a super quick tutorial. I will not cover all details. Let's go through the overview.

# Boot to Rust

- The first step is to boot into the hypervisor.
- And here's how to do it in Rust.
- The `boot` function is the entry point of the hypervisor.
- It sets up the stack pointer, and jumps to the `main` function.
- Inline assembly in Rust is similar to C.

# Putchar via SBI

- The next steps would be having a printf.
- In RISC-V, SBI, which is a standard interface similar to UEFI, can be used to print a character.
- Inline assembly is a bit more complicated, but it's simple. It defines input and output registers, and execute "ecall" instruction to call the firmware providing SBI.

# `println!`

- Now we can print a character.
- Next, let's print a string like printf in C.
- In Rust, we can use standard library to format strings, and the minimal implementation looks like this.
- It defines a struct implementing `Write` trait, print characters if `write_str` is called, and define a `println` macro.

# Dynamic memory allocator

- Another thing we need to prepare for Rust is - dynamic memory allocator.
- Here's a naive bump allocation algorithm.
- Similar to `println!` macro, define a struct, implement `GlobalAlloc` trait for it, and define it as the global allocator.
- Once we have a global allocator, we can collections and containers like vector, string, and hash map.

# Guest memory

- OK so we're ready for bare-metal Rust.
- Let's go back to the hypervisor.
- For hypervisor, the first thing would be to prepare guest memory space.
- TODO:

# Guest page table

- TODO:

# Load the kernel image

- Now we have a guest memory.
- Let's fill it with the boot image.
- Linux kernel defines a raw binary image format for RISC-V.
- All we need is to verify raw binary image, and copy the image into the base address.

# Enter the guest mode

- Now we're ready to boot the virtual machine.
- In RISC-V it's way simpler than other CPUs: write the guest state to registers, and execute the `sret` instruction.
- Entering the guest mode is very CPU specific but all CPUs do the same thing: load the guest state and start the guest execution.

# Handle VM eixsts (1/2)

TODO:

# Handle VM eixsts (2/2)

TODO:

# VM Exit (1/2): Hypervisor call

TODO:

# VM Exit (2/2): Memory-mapped I/O

TODO:

# More on Web!

TODO:

# Invest in user (*"developer"*) experience

- This is the last slide. I've shown you some examples of higher-level interfaces like: new virtio, and new hypercalls.
- But, going higher abstraction does not always mean inventing new interfaces.

- That is, designing the user, or "developer" interface is also important.
- An example is "hypervisor as a library", it is a blog post I wrote recently.

- Let's look at this example. To use QEMU, you will specify: CPU architecture, kernel image, the disk image, and so on.
- It's straight-forward and is easy to use.

- However, what if ... you can use the Linux virtual machine like this?
- It's mostly the same API we use to start a subprocess in Rust.
- There's no new technologies. It's just a wrapper of low-level QEMU commands.

- I've seen similar patterns in Web frontend frameworks like React.
- That is, make people fall in love with your hypervisor or your project.
- Investing in the user interface is key, and it's very hard than you anticipate.

- I'm looking forward to seeing your next great hypervisor with a great user experience.
