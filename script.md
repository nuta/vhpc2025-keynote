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

- Guest memory consists of regions.
- Each region has the base guest address, the size, and the type of the region like: free RAM area that guest may use, or memory-mapped I/O area for virtual devices.

# Guest page table

- To map the guest memory regions into the guest physical address space, we need a guest page table.
- It's very similar to a page table, but instead of virtual address, it translates guest physical address.

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

- So we're now in the guest mode.
- CPU keeps executing instructions in the guest mode as usual, and when it needs an assistance from the hypervisor, it jumps to the hypervisor (or host) kernel, which is called "vm exit".

- In RISC-V, VM exit is implemented as a trap. `stvec` register contains the trap handler address, and the handler will be called when a VM exit occurs.
- The trap handler saves the guest state such as general-purpose registers, and jumps to the handler function.

# Handle VM eixsts (2/2)

- In this slide, few things are defined for later slides.
- The main job of the trap handler is to handle the VM exit.
- In this step, the handler does nothing but panics. But in general, it receives the details of the VM exit, and do something with it, and continue the guest again.

- So how VM exit handling looks like?

# VM Exit (1/2): Hypervisor call

- The first example is a hypervisor call.
- This example is in RISC-V, but the pattern is the same for other CPUs.
- `scause` register contains the reason of the trap, or VM exit.
- We can use it to detect the hypervisor call.
- Here we emulate the console putchar interface in SBI.
- And once it's done, update the program counter to continue from the next instruction, and go back to the guest mode.

# VM Exit (2/2): Memory-mapped I/O

- Memory-mapped I/O is similar to hypervisor call, but it's more complex.
- Memory-mapped I/O appears as a page fault in the trap handler.
- When the guest page fault happens, the trap handler receives more details of the page fault, such as guest address, and the instruction at that time.
- Then, the handler search the guest memory regions to find the corresponding region, if the fault address is in a memory-mapped I/O area, it emulates the device.

# More on Web!

TODO:

# A life of a hypervisor

- Let's summarize what we've seen so far.
- To put it simply in JavaScript, the life of a hypervisor looks like this.
  1. Prepare guest memory.
  2. Load the boot image.
  3. Initialize the vCPU state.
  4. Enter the guest mode.
  5. Handle VM exits and go back to the guest mode.

- This means that the guest keeps running until it needs an assistance from the hypervisor.
- In other words, hypervisors are the `catch` block.

# Hypervisors are the `catch` block (mostly)

- Here's the flow of the hypervisor.
- It enters guest, exit to the host, and the hypervisor catches the VM exit, and continue the guest.

- This is similar to interpreters. In JavaScript, we run a program in a try block, the interpreter keeps running the program, and when the program can't continue, we catch an exception.

# Hypervisors are the `catch` block (mostly)

- That is, the guest mode is like a try block.
- The hypervisor's trap handler is like a catch block.
- VM exit can be considered as a throw.
- And the details of VM exit are like the exception thrown by the guest.

# Real-world virtualization APIs

- Let's look at some real-world virtualization APIs to observe the same try-catch pattern.

# Linux KVM API

- The first example is Linux KVM.
- It's a file-based interface.
- Open a special KVM file, do some ioctls to create virtual CPUs, run ioctl to enter the guest mode, and it returns when VM exits.
- FreeBSD also has a similar interface.

# Fuchsia API

- The second example is Fuchsia, an operating system developed by Google.
- It's capability-based, and have its own interface.
- API names are different but the pattern is very similar to Linux KVM.
- Initialize the guest memory, create virtual CPUs, enter the guest mode, and handle VM exits.

# seL4 libvmm API

- The third example is seL4, a microkernel-based operating system.
- Unlike Linux KVM, seL4's vmm library API is callback-based.
- Initialize the hypervisor in `init` function, and when VM exits, `fault` function is called, and do memory-mapped I/O emulation.
- It looks different from KVM, but the pattern is the same: initialize, enter the guest mode, catch the VM exit, and continue.

# Hypervisors are `catch` blocks (w/ continuation)

- We saw the following pattern in all APIs:
  1. Initialization. Prepare guest memory and vCPUs.
  2. Try. Enter the guest mode.
  3. Catch. Handle an exception such as memory-mapped I/O emulation.
  4. Continue. Go back to the guest mode.
- TODO:
- So hardware-assisted virtualization is not really about emulating hardware, or virtual machines. It's a generic mechanism to run a program in a different world.
  - This means hardware-assisted virtualization has great potential to be applied to more areas, not only virtual machines.

# The `try-catch` pattern in action

- Next, let's look at some applications of this hardware-assisted try-catch, beyond virtual machines.

# gVisor: Hypervisor as a container sandbox

- gVisor is a hypervisor designed for running Linux containers in a strong isolation for multi-tenancy.
- gVisor supports a few different sandboxing mechanisms, and one of them is a KVM based isolation.
- In Guest mode, gVisor runs its own applicaiton kernel written in Go. It's Linux-compat, but it's not a full Linux kernel.
- By having a KVM, system calls are trapped and emulated.
- This reduces the attack surface.

# Hyperlight: Hypervisor as a function sandbox

- Hyperlight is another sandbox solution by Microsoft.
- It's similar to gVisor, but it's designed for functions, not Linux containers.

- Hyperlight provides a strong isolation boundary for multi-tenant services.
- It's also based on virtualization technology, like Linux KVM.

- gVisor implements Linux system calls, but in Hyperlight, the host provides application-specific hypercalls.

# Noah: Hypervisor for system call emulation

- The next example is Noah.
- Noah is a Linux system call emulator based on the virtualization technology.
- In guest mode, it doesn't have a guest kernel. Instead, system calls cause VM exits, and the host process emulates Linux system calls.
- So Noah uses the virtualization technology as a system call hook to catch system calls.

# Nabla Containers: Higher-level hypervisor interface

- The last example is Nabla Containers.
- In Nabla Containers, the hypervisor does not provide Virtio,it doesn't provide virtual devices.
- Instead, it provides a high-level interface to the guest.

- Here's the comprehensive list of the hypercalls Nabla provides.
- Each hypercalls corresponds to a system call in Linux.
- For example, reading a block device would be translated to reading the disk image in the host OS.

- Nabla demonstrates that if we don't need to care about the compatibility, we have a lot of room to design the guest interface.

# (hardware-assisted) hypervisors look like interpreters

- To summarize, hardware-assisted virtualization is a generic mechanism to run a program in a different world.
- This is similar to interpreters such as JavaScript and Python engines.
 - Both have a different world.
 - Both define a clear interface between the guest and the host.
 - And Both also provides a secure isolation boundary to run untrusted code.

# That is ... virtualization is not only for VMs!

TODO:

# What's coming next? - The guest ↔︎ host interface goes higher!

- I think we'll see more higher-level interfaces in the future.
- Let me share my favorite ideas.

# Virtio alternatives: Do we need a queue interface?

- Virtio is the de facto standard for the guest-host interface.
- It's a generic queue and can be used for many different purposes such as network device, block device, GPU, file system, and more.

- However, do we really need a queue interface?
- It's similar to Linux system call design. Linux has an interface similar to virtio called io_uring, but it's just one of the Linux interface designs.

- We can design a new interface. We don't have to be limited by the Virtio.

# Higher-level interfaces in Virtio: virtio-tcp & udp

- Virtio already provides high-level interfaces.
- For example, file system is abstracted as virtio-fs, based on the file system in userspace.

- However, we don't have high-level abstraction for network sockets such as TCP and UDP.
- This is a bit challenging because typically TCP/IP implementation does not expect to offload the TCP/UDP processing to the hypervisor.

- Interestingly, we have a previous work.
- libkrun is a dynamic library to isolate a process into the guest mode.
- It provides seamless integration with the host environment.
- One of its integrations is a network stack. Called Transparent Socket Impersonation.

- I think the similar high-level socket interface might be possible in other hypervisors.

# Strongly-isolated JavaScript

- The next idea is to focus on the application, not the hypervisor.
- This is similar to Hyperlight.
- Here I brought up JavaScript.
- Why JavaScript? Not just because it's a popular programming language, but because it's creeping into the backend world, where multi-tenancy technologies like hypervisors are required.

- Today, vedors are building JavaScript-specific clouds, and they are using V8 isolates as a lightweight security boundary.

- However, V8 isolates are not secure enough.
- Also, hypervisors can be lightweight.

- Designing hypervisor-based JavaScript runtime is not obvious, but if it happens, it could be a good product.

# AI agent sandboxing: LLM↔︎guest interface design

- We're in the age of AI. So let's take a look at AI agent sandboxing.
- It's becoming a hot topic in the AI agent industry.
- That is, everybody need hypervisors, not just running virtual machines.

- The question is: what should the hypervisor provide to the AI agent?
- It's still unclear because the trend is changing so fast.

- But what I expect is people won't be satisfied with the current offering from the cloud providers.
  - We might need a fast snapshot technology to make AI iterations faster.
  - We might need a high-level interface for the AI agent to control the sandbox.

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

# Conclusion

TODO: