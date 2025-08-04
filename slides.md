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

# Hypervisors are the `catch` block

---

# Linux KVM

```c
```

---

# FreeBSD bhyve

```c
```

---

# Fuchsia

(^ hardware-assisted)

```c
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
