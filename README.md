[![Build Status](https://github.com/rust-osdev/ps2-mouse/workflows/Build/badge.svg)](https://github.com/rust-osdev/ps2-mouse/actions?query=workflow%3ABuild) [![Docs.rs Badge](https://docs.rs/ps2-mouse/badge.svg)](https://docs.rs/ps2-mouse/)

# ps2 mouse
This crate provides a basic interface for interacting with a ps2 mouse.

## Basic Example
```rust
use ps2_mouse::{Mouse, MouseState};
use spinning_top::Spinlock;
use x86_64::instructions::port::PortReadOnly;

pub static MOUSE: Lazy<Spinlock<Mouse>> = Lazy::new(|| Spinlock::new(Mouse::new()));

// Initialize the mouse and set the on complete event.
fn init_mouse() {
    MOUSE.lock().init().unwrap();
    MOUSE.lock().set_on_complete(test);
}

// This will be fired when a packet is finished being processed.
fn on_complete(mouse_state: MouseState) {
    println!("{:?}", mouse_state);
}

// An example interrupt based on [https://os.phil-opp.com/hardware-interrupts/]. The ps2 mouse is configured to fire
// interrupts at PIC offset 12.
extern "x86-interrupt" fn mouse_interrupt_handler(_stack_frame: &mut InterruptStackFrame) {
    let mut port = PortReadOnly::new(0x60);
    let packet = unsafe { port.read() };
    MOUSE.lock().process_packet(packet);

    unsafe {
        PICS.lock()
            .notify_end_of_interrupt(InterruptIndex::Mouse.into());
    }
}
```
