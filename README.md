# Super FSM

### "The 'Super Loop + FSM' architecture, powering 90% of embedded systems, deserves higher standards!"

Classic FSM. Done right.

Super FSM elevates the switch-case state machine with 15 professional disciplines
for production-grade embedded C — no RTOS required.

---

## The Problem

Every embedded developer knows this pattern:

```c
switch (state) {
    case ST_IDLE:
        if (event == EVT_START) state = ST_RUNNING;
        break;
    case ST_RUNNING:
        // grows to 200 lines over time
        // no on_enter, no on_exit, no timeout
        // ISR writes directly to state variable
        // race condition waiting to happen
        break;
}
```

It works. Until it doesn't.

---

## The Solution

Super FSM keeps the familiar switch-case structure but applies 15 professional
disciplines that make it production-ready:

| Rule | Discipline |
|------|-----------|
| 1 | Each module owns its FSM — no monolithic state machine |
| 2 | No blocking code inside state functions |
| 3 | All transitions go through a single `transition()` function |
| 4 | Guaranteed `on_enter` / `on_exit` on every state change |
| 5 | Each state is a separate function — switch only dispatches |
| 6 | Invalid events are logged, never silently ignored |
| 7 | All FSM data lives in a single context struct |
| 8 | Every waiting state has a timeout |
| 9 | Super loop follows strict Input → Process → Output order |
| 10 | ISRs only set flags — FSM runs in main context |
| 11 | Previous state always recorded (`prev` field) |
| 12 | State and event name tables for readable debug output |
| 13 | Transition table documented alongside the code |
| 14 | HAL abstraction makes FSM testable on PC |
| 15 | FSMs communicate via events, never read each other's state |

---

## Quick Start

```c
/* 1. Define your states and events */
typedef enum { ST_CLOSED, ST_OPEN, ST_LOCKED, ST_ALARM } state_t;
typedef enum { EVT_OPEN, EVT_CLOSE, EVT_LOCK, EVT_TAMPER, EVT_RESET } event_t;

/* 2. Define your context struct (Rule 7) */
typedef struct {
    state_t  state;
    state_t  prev;          /* Rule 11 */
    uint8_t  alarm_count;
    uint32_t timeout_ticks; /* Rule 8  */
} door_ctx_t;

/* 3. Run the FSM in your super loop */
while (1) {
    event_t evt = read_inputs();    /* Rule 9: Input  */
    door_fsm(&ctx, evt);            /* Rule 9: Process */
    write_outputs(&ctx);            /* Rule 9: Output  */
    door_tick(&ctx);                /* Rule 8: Timeout */
}
```

Build and run the door example on any machine:

```bash
git clone https://github.com/erol-design/Super-FSM
cd super-fsm
make
./build/door_example
```

---

## Project Structure

```
super-fsm/
├── core/
│   ├── fsm.h          # Framework API
│   └── fsm.c          # Framework implementation
├── examples/
│   └── door/
│       ├── door_fsm.c # All 15 rules applied
│       └── main.c     # PC simulation (no hardware needed)
├── ports/
│   └── stm32/
│       └── hal_port.h # STM32 HAL abstraction layer
└── docs/
    └── 15-rules.md    # Detailed rule reference
```

---

## Design Principles

**No dynamic memory.** Every context is statically allocated.

**No RTOS dependency.** Runs on bare-metal super loop.

**HAL abstraction.** FSM logic never touches registers directly —
swap `hal_port.h` to run on STM32, ESP32, or your PC.

**Testable.** The door example compiles and runs on any machine
with `gcc`. No hardware, no debugger, no problem.

---

## Who Is This For

- Embedded developers who outgrew the basic switch-case FSM
- Teams looking for a consistent, reviewable FSM pattern
- Anyone building reliable bare-metal firmware

---

## License

MIT — use it, modify it, ship it.

---

*Super FSM is the companion code for the book*
*"Super Embedded System" by [Erol YILMAZ].*
