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

- Intel VT
- AMD SVM
- Arm Virtualization Host Extensions
- RISC-V H extension

::right::

TODO: image here
 

---
layout: center
---

Intel VT/AMD SVM/Arm VHE/RISC-V H are ...

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
- FreeBSD bhyve uses a similar interface (`ioctl`).

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

# seL4 libvmm API (simplified)

```c
void init(void) {
  // 1-2. Prepare memory and load the program
  linux_setup_images(...);
  // 3-4. Initialize vCPU and start it
  guest_start();
}

// 5. Handle exit and go back to guest mode
seL4_Bool fault(size_t vcpu_id, microkit_msginfo msginfo, ...) {
  label = microkit_msginfo_get_label(msginfo);
  switch (label) {
  case seL4_Fault_VMFault:
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

# gVisor: Hypervisor as a container sandbox

- A VMM for running multi-tenant Linux containers.
- A new guest kernel written in Go emulates Linux.
- Guest ↔︎ host interface is limited than running containers directly.

gVisor: https://github.com/google/gvisor

---

# Hyperlight: Hypervisor as a function sandbox

- A VMM for running multi-tenant functions (vs. containers in gVisor).
- Uses hardware-assisted virtualization as an isolation boundary.
- Fast cold start time (1-2 ms).
- `Out32` to trigger VM exit.

Hyperlight: https://github.com/hyperlight-dev/hyperlight

----

# Noah: Hypervisor for system call emulation

- Run applications in guest mode **w/o guest kernel**.
- VMM intercepts system calls and emulates them.
- Hypervisor as a system call hook.

https://doi.org/10.1145/3381052.3381327

---
layout: two-cols-header
---

# Nabla Containers: Higher-level hypervisor interface

:: left ::

- Unikernels as (strongly-isolated) processes.
- Q: Do we really need virtual "devices" in guest?
- Hypercalls look more like system calls, not devices.

:: right ::

```
walltime
puts
poll
blkread
blkwrite
netwrite
netread
halt

blkinfo (deprecated)
netinfo (deprecated)
```

<!-- Unikernels as Processes: https://doi.org/10.1145/3267809.3267845 -->

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
