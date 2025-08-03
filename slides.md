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
---

## Writing a Hypervisor From Scratch

Seiya Nuta

<nuta@seiya.me>

---
# What I'm going to talk about

- ❌ Writing a hypervisor from scratch in 45 minutes

- ❌ Being able to write a hypervisor from scratch

- ✅ Rediscover the not-so-obvious perspective on virtualization: **virtualization are not only about implementing fast virtual machines!**

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

# 

---
layout: cover
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
layout: cover
---

# Hypervisors implement the `catch` block

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
layout: fact
---

# Hello

woorld

---
#

---
# Introduction

DO we need a hypervisor?

DO we need a hypervisor?

DO we need a hypervisor?

DO we need a hypervisor?

DO we need a hypervisor?

asd
