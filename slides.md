---
theme: default
title: Writing a Hypervisor From Scratch
info:
class: text-center
drawings:
  persist: false
transition: false
mdc: true
seoMeta:
  ogImage: auto
fonts:
  mono: JetBrains Mono
---

## Writing a Hypervisor From Scratch

Seiya Nuta

<nuta@seiya.me>

---
layout: center
---

# What I'm going to talk about

- ❌ Writing a hypervisor from scratch in 40 minutes

- ❌ Being able to write a hypervisor from scratch

- ✅ (hardware-assisted) virtualization are not only for VMs!

---
layout: two-cols-header
---

# Hardware-assisted virtualization

::left::

- Intel VT-x
- AMD-V
- Arm Virtualization Host Extensions
- RISC-V H extension

::right::

TODO: image here
 

---
layout: center
---

Intel VT-x/AMD-V/Arm VHE/RISC-V H are ...

# hardware-assisted `try-catch` (with continuation)

---

# Try-catch pattern

```ts
try {
    emoji = readFile("emoji.txt");
} catch (error) {
    panic(`failed to read a file: ${error}`);
}
```

- The program keeps running until it encounters an exception.
  - i.e. `catch` is called only when the program needs a help.
- You can't resume the program from the point of failure.
  - Retry from the beginning or abort the program.

---

# Try-catch pattern + continuation

Let's assume that we have a `resume` callback:

```ts
try {
    emoji = readFile("emoji.txt");
} catch (error, resume) {
    if (error instanceof FileNotFoundError) {
        resume("🍎");
    }

    panic(`failed to read a file: ${error}`);
}
```

- cf. Algebraic Effects

---
layout: fact
---

---
src: ./pages/hypervisor-in-1000-lines.md
---

---

# A life of a hypervisor (in JavaScript)

```ts
kernelImage = readFileSync("kernel.bin");

memory = new GuestMemory();  // 1. Prepare memory
memory.add(kernelImage);     // 2. Load the program (guest kernel)

vcpu = new VCpu();           // 3. Initialize vCPU state
for (;;) {
    try {
        vcpu.enterGuest();   // 4. Enter the guest mode
    } catch (exit) {
        handleVMExit(exit);  // 5. Handle exit and go back to 4
    }
}
```

---

# Hypervisors are the `catch` block!

| JavaScript | Hardware-assisted virtualization |
| --- | --- |
| `try` block | guest mode |
| `catch` block | hypervisor's trap handler |
| `throw` | VM exit |
| `error` | VM exit reason |

---

# Linux KVM API (simplified)

```c
guest_memory = mmap(...);                   // 1. Prepare memory
memcpy(guest_memory, ...);                  // 2. Load the program

kvm_fd = open("/dev/kvm", O_RDWR);          
vm_fd = ioctl(kvm_fd, KVM_CREATE_VM, 0);    
vcpu_fd = ioctl(vm_fd, KVM_CREATE_VCPU, 0); // 3. Initialize vCPU state

struct kvm_run *run = mmap(...);
for (;;) {
    ioctl(vcpu_fd, KVM_RUN, &run);          // 4. Enter the guest mode
    switch (run.exit_reason) {              // 5. Handle exit and
        case KVM_EXIT_MMIO:                 //      go back to (4)
        ...
    }
}
```

---

# FreeBSD bhyve API (simplified)

TODO: Fix APIs

```c
ctx = vm_create("vm_name");
vm_setup_memory(ctx, mem_size, VM_MMAP_ALL); // 1. Prepare memory
guest_mem = vm_map_gpa(ctx, addr, size);     // 2. Load the program
memcpy(guest_mem, ...);
vm_vcpu_reset(ctx, 0);                       // 3. Initialize vCPU
vm_set_register(...);

struct vm_exit vmexit;
for (;;) {
    vm_run(ctx, 0, &vmexit);                 // 4. Enter guest mode
    switch (vmexit.exitcode) {               // 5. Handle exit and
        case VM_EXITCODE_MMIO:               //       go back to (4)
        ...
    }
}
```

---

# Fuchsia API (simplified)

```c
zx_guest_create(..., &guest, &vmar);
zx_vmar_allocate(..., &addr);        // 1. Prepare memory
memcpy(addr, ...);                   // 2. Load the program
zx_vmar_map(vmar,...);

zx_vcpu_create(guest, ..., &vcpu);   // 3. Initialize vCPU
zx_vcpu_write_state(vcpu, ...);

zx_port_packet_t packet;
for (;;) {
    zx_vcpu_enter(vcpu, &packet);    // 4. Enter guest mode
    switch (packet.type) {           // 5. Handle exit and
        case ZX_PKT_TYPE_GUEST_MEM:  //       go back to (4)
        ...
    }
}
```

---

# Hypervisors are `catch` blocks

- Virtual Machine Monitor (VMM)

---
layout: cover
---

# The `try-catch` pattern in action

---

# Hyperlight (2024)

Microsoft Azure Core Upstream team

- Out32


https://opensource.microsoft.com/blog/2024/11/07/introducing-hyperlight-virtual-machine-based-security-for-functions-at-scale/

---

# Nabla Containers

---

# gVisor


---

# Noah

---

# Virtulization is not only about virtual machines

- Interface matters.
- Consider (hardware-assited) hypervisors as JavaScript interpreters.
  - More application-specific interfaces.
    - Reducing "semantic gap", less overheads, and offloading to stuff to hypervisor. Like TLS.
  - How do you interact with JavaScript world? Shared memory, queues, or calling host functions like PIO (`outb`/`inb`)?
- Do we really need virtio?
  - You can implement a hypervisor on top of a microkernel.
- If you see like this, don't Hyperlight, gVisor look familar to you?

---
layout: cover
---

# What's coming next? (my ideas)

---

# Virtulization-based isolation in microkernels? (needs investigation!)
