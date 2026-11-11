# Round-Robin Arbiter UVM Verification Project

This repository contains a complete **SystemVerilog** project demonstrating the **design and verification** of a parameterizable **Round-Robin Arbiter (DUT)**.  
It includes a synthesizable RTL module and a comprehensive **UVM (Universal Verification Methodology)** testbench built entirely from scratch.

This project serves as a **hands-on, industry-standard example** of verifying a common hardware control module — focusing on **protocol checking**, **state verification**, and **functional coverage**.

---

## Features

### RTL Design (DUT)

- **File:** `rr_arbiter.sv`  
- **Parameter:** `N_CLIENTS` (default = 4) — defines number of clients.
- **Functionality:** Grants access to one requesting client at a time using a **round-robin priority scheme** to ensure fairness.

**Protocol-Compliant:**
- Correctly implements round-robin priority update logic.  
- Ensures the `grant` output is **one-hot** (or all zeros if no requests).  
- Guarantees a grant is only issued to a client that is **actively requesting**.

---

### UVM Testbench Components

#### `rr_arbiter_transaction`
- Defines one observation cycle, capturing `req` and `grant` bus values.
- Parameterized by `N_CLIENTS`.

#### `rr_arbiter_agent` (Active Master)
Encapsulates the following:
- **`rr_arbiter_driver`** — Drives randomized `req` stimulus each clock cycle.  
- **`rr_arbiter_monitor`** — Observes `req` and `grant`, includes functional coverage.  
- **`rr_arbiter_sequencer`** — Manages flow of sequences to the driver.

#### `rr_arbiter_scoreboard`
- Receives transactions from the monitor.  
- Maintains a **shadow model** of arbiter’s state (`last_grant_idx`).  
- Performs protocol checks:
  - Grant is one-hot or zero.
  - Grant issued only if requested.
  - Predicts expected grant and compares it to DUT output.
  - Reports `UVM_ERROR` on mismatches.

#### `rr_arbiter_env`
- Top-level UVM component instantiating agent and scoreboard.  
- Parameterized by `N_CLIENTS`.

#### Sequences & Tests
- Library of **directed** and **randomized** tests.
- Non-parameterized wrappers provided for easier invocation.

---

## Project Structure



```bash
rr-arbiter-uvm-verification/
├── README.md
├── rtl/
│ └── rr_arbiter.sv
├── tb/
│ ├── rr_arbiter_if.sv
│ ├── tb_top.sv
│ │
│ ├── uvm_env/
│ │ ├── rr_arbiter_transaction.sv
│ │ ├── rr_arbiter_sequencer.sv
│ │ ├── rr_arbiter_driver.sv
│ │ ├── rr_arbiter_monitor.sv
│ │ ├── rr_arbiter_agent.sv
│ │ ├── rr_arbiter_scoreboard.sv
│ │ ├── rr_arbiter_env.sv
│ │ └── rr_arbiter_env_pkg.sv
│ │
│ ├── seq_lib/
│ │ ├── rr_arbiter_directed_seq.sv
│ │ ├── rr_arbiter_random_seq.sv
│ │ └── seq_lib_pkg.sv
│ │
│ └── tests/
│ ├── rr_arbiter_directed_test.sv
│ ├── rr_arbiter_random_test.sv
│ ├── rr_arbiter_directed_test_4.sv
│ ├── rr_arbiter_random_test_4.sv
│ └── tests_pkg.sv
│
└── sim/
└── Makefile
```


---

## Requirements

- **SystemVerilog-capable simulator** supporting UVM  
  (e.g., *Synopsys VCS*, *Cadence Xcelium*, *Siemens Questa*).  
- **Linux environment** to run `make`.  
- *(Optional)* **Waveform viewer** — e.g., `DVE`, `gtkwave`.  
- *(Optional)* **Coverage tools** — e.g., `urg`, `Verdi` for coverage visualization.

---

## How to Run

All commands are run from the `sim/` directory.

```bash
# Navigate to simulation directory
cd sim/

# 1. Clean previous builds
make clean

# 2. Compile and run default test (rr_arbiter_random_test_4)
make

# OR run explicitly
make TESTNAME=rr_arbiter_random_test_4


## Checking Results

A test is considered **PASSED** when the UVM report summary shows **no errors or fatals** at the end of the simulation log.

After running a test, view the log file:

```bash
cat run.log
```

You should see output similar to the following:

```bash
# --- UVM Report Summary ---

# ** Report counts by severity **
# UVM_INFO    :   ...
# UVM_WARNING :   0
# UVM_ERROR   :   0   <-- SUCCESS
# UVM_FATAL   :   0

```

If `UVM_ERROR` or `UVM_FATAL` is non-zero, the test has failed and requires debugging.

## Running other tests

You can specify a different test using the `TESTNAME` variable in the make command.

```bash
# Example: Run the randomized test
# Run directed test for N=4
make TESTNAME=rr_arbiter_directed_test_4
```

To re-run the default base test:

```bash
make TESTNAME=rr_arbiter_base_test
```

The simulation will automatically compile, execute, and generate a report summary at the end of the log.

## Generating Functional Coverage Reports

The Makefile supports coverage collection and reporting.

```bash
# Compile, Run, and Generate Coverage Report
make coverage
# OR
make report
```

## Debugging Waveforms

The simulation automatically generates a **waveform dump file** that captures all signal activity during the test.  

| **Simulator** | **Waveform File** | **Viewer Example** |
|----------------|-------------------|--------------------|
| **Synopsys VCS** | `dump.vcd` | `gtkwave dump.vcd &` |
| **Siemens Questa / ModelSim** | `waves.shm` | `vsim -view waves.shm &` |

### Example (VCS + GTKWave)

```bash
# Open the waveform file
gtkwave dump.vcd &
```

## Test Plan Summary

The project includes two primary tests designed to verify both **basic functionality** and **randomized protocol compliance** of the AXI4-Lite slave DUT.

| **Test Name** | **Sequence Used** | **Type** | **Description** |
|----------------|-------------------|-----------|-----------------|
| `rr_arbiter_directed_test_4` | `rr_arbiter_directed_seq_4` | Directed | Sends specific request patterns to validate grant logic, priority updates, and wrap-around behavior.|
| `rr_arbiter_random_test_4` | `rr_arbiter_random_seq` | Randomized | Executes 100 cycles of random requests to stress-test arbiter under varied load conditions. |

### Verification Goals

- **Validate protocol compliance**
  - Grant is one-hot or zero.
  - Grant is only issued when a client is actively requesting.

- **Verify correct priority updates**
  - Round-robin scheme is strictly followed across cycles.

- **Test various request scenarios**
  - No requests (all zeros)
  - Single active request
  - Multiple simultaneous requests

- **Achieve high functional coverage**
  - Cover all request and grant pattern combinations.
  - Include coverage for all priority states.
  - Cross coverage between key conditions (e.g., request/grant/priority transitions) via the monitor’s covergroup.

---

### Scoreboard Checks

The `rr_arbiter_scoreboard` performs automated checks every cycle to ensure correctness:

- Checks if the grant bus is **one-hot or zero**.
- Ensures that `grant[i]` implies `req[i]` was asserted.
- Predicts the **expected grant** based on the internal model of `last_grant_idx` and the current `req`.
- Compares the **predicted grant** to the DUT’s **actual grant**.
- Any mismatch immediately triggers a `UVM_ERROR`.

---

## Future Enhancements

- Add configurable arbitration modes such as:
  - **Fixed Priority**
  - **Weighted Round-Robin**
- Implement **corner-case sequences** to test specific timing and state wrap-around conditions.
- Add **cover directives** or **SystemVerilog assertions (SVA)** in RTL for formal verification.
- Integrate **error injection sequences**, such as invalid or stuck request signals.
- Extend coverage to capture **timing transitions** and **state change sequences**.

---

## Author

**Pranav Asuri**  

---
