---
layout: cover
---

# Writing a hypervisor from scratch (super quick walkthrough!)

---

# Target

- Rust programming language (`no_std` mode)
- 64-bit RISC-V with "H"ypervisor extension
- QEMU
  - `qemu-system-riscv64 -machine virt -cpu rv64,h=true`

<br>

- **Note:** Details are omitted for brevity.

---

# Boot to Rust

```rs
pub extern "C" fn boot() -> ! {
    unsafe {
        asm!(
            "la sp, __stack_top", // Stack address (from linker script)
            "j {main}",           // Jump to main.
            main = sym main,
            options(noreturn)
        );
    }
}

fn main() -> ! {
    loop {} // Infinite loop.
}
```

---

# Putchar via SBI

```rs
fn sbi_putchar(ch: u8) {
    unsafe {
        asm!(
            "ecall",
            in("a6") 0, // SBI function ID
            in("a7") 1, // SBI extension ID (Console Putchar)
            inout("a0") ch as usize => _, // Argument #0
            out("a1") _ // Argument #1 (not used)
        );
    }
}
```

---

# `println!` macro (Rust's `printf`)

<style>
code {
  font-size: 1.4em;
}
</style>

```rs
pub struct Printer;

impl core::fmt::Write for Printer {
    fn write_str(&mut self, s: &str) -> core::fmt::Result {
        for byte in s.bytes() {
            sbi_putchar(byte);
        }
        Ok(())
    }
}

macro_rules! println {
    ($($arg:tt)*) => {{
        use core::fmt::Write;
        let _ = writeln!($crate::Printer, $($arg)*);
    }};
}
```

---

# Dynamic memory allocator ("bump" algorithm)

<style>
code {
  font-size: 1.34em;
}
</style>

```rs
pub BumpAllocator {
    next: usize,
    end: usize,
}

unsafe impl GlobalAlloc for BumpAllocator {
    unsafe fn alloc(&self, layout: Layout) -> *mut u8 {
        let start = self.next;
        self.next += align_up(start, layout.align());
        assert!(self.next <= self.end, "out of memory");
        start as *mut u8
    }
}

#[global_allocator]
pub static GLOBAL_ALLOCATOR: BumpAllocator = BumpAllocator { ... };
```

- Unlocks vector (`Vec`), `String`, `LinkedList`, `HashMap`, and more.


---

# 