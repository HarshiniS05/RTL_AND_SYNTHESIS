# Week 1 - Day 2

## 📌 Introduction
In Day 2 of the workshop, we explored the role of **cell libraries (`.lib` files)** in synthesis, the difference between **hierarchical vs flat synthesis**, and how to perform **submodule-level synthesis**.  
We also studied **various flip-flop coding styles** (with asynchronous/synchronous resets and sets) and synthesized them using **Yosys**.  
These concepts are fundamental in bridging RTL design with real hardware implementations.

---

## 🎯 Learning Objectives
By the end of Day 2, I learned to:
- Understand the purpose and contents of `.lib` files.  
- Perform **hierarchical vs flat synthesis** and compare their advantages.  
- Apply **submodule-level synthesis** to large designs.  
- Implement and verify **different flip-flop coding styles** in Verilog.  

---

## 1. Library Files (`.lib`)

A `.lib` file is a **cell library file** used during synthesis.  
It contains information about:
- Standard cells and their variations  
- Features such as leakage power, voltage (V), current (I), and delay  

### Command to Open `.lib` File:
```bash
gedit /home/harshini/VLSI/sky130RTLDesignAndSynthesisWorkshop/lib/sky130_fd_sc_hd__tt_025C_1v80.lib
```

![dot lib](images/dot_lib.png)  
*The `.lib` file provides power, delay, and functionality details of each standard cell.*

---

## 2. Hierarchical vs Flat Synthesis

| Feature             | Hierarchical Synthesis | Flat Synthesis |
|---------------------|-------------------------|----------------|
| **Structure**       | Preserves module hierarchy | Flattens everything into one |
| **Debugging**       | Easier (clear separation) | Harder (all merged) |
| **Optimization**    | Moderate                | Higher |
| **Reusability**     | High (good for repeated modules) | Low |
| **Preferred Use**   | Large designs with multiple instances | Performance-critical designs |

---

## 3. Hierarchical Synthesis Example

### Commands:
```bash
gedit multiple_modules.v
yosys
read_liberty -lib path/to/library.lib
read_verilog multiple_modules.v
synth -top multiple_modules
abc -liberty path/to/library.lib
show
```

![multiple module code](images/multiple_module_code.png)  
*Verilog code for multiple modules.*

![synth output](images/multiple_module_u12.png)  
*Synthesized output showing preserved hierarchy.*

Generate netlist:
```bash
write_verilog multiple_modules_hier.v
!gedit multiple_modules_hier.v
```

![Hierarchical Netlist File](images/netlist_multiple_module_hierv.png)  
*Netlist of the hierarchical design it has a sub module.*

---

## 4. Flat Synthesis Example

```bash
read_verilog multiple_modules.v
synth -top multiple_modules
abc -liberty path/to/library.lib
flatten
show
```

![synth Flat](images/mm_flatten_show.png)  
*Netlist output after flattening the design which does not have the u1 &u2 submodule .*

---

## 5. Submodule Level Synthesis

### Why Use Submodule Synthesis?
- Useful when multiple instances of the same module exist.  
- Saves design effort and improves scalability.  
- Commonly used in large CPU/SoC designs.  

### Commands:
```bash
yosys
read_liberty -lib path/to/library.lib
read_verilog sub_module1.v
abc -liberty path/to/library.lib
show
```

![Synth-Submodule](images/mm_submodule1.png)  
*Synthesis at submodule level and it's netlist.*

---

## 6. Various Flip-Flop Coding Styles

### Why Do We Need Flops?
- Pure combinational circuits create glitches.  
- Flip-flops **store values** and provide stable outputs.  
- Without initialization, circuits evaluate garbage → hence we use **set/reset**.  

*Types:*  
- **Asynchronous reset** → reset works anytime, independent of clock.  
- **Synchronous reset** → reset only with clock edge.  

---

### Example 1: D Flip-Flop with Asynchronous Reset

```bash
iverilog dff_asyncres.v tb_dff_asyncres.v
./a.out
gtkwave tb_dff_asyncres.vcd
```

![DFF Async Reset Waveform](images/tb_dff_asyncres.png)  
*Waveform of DFF with async reset.Normal Operation (without reset):*

__Normal Operation (without reset):__

When the reset signal is inactive and goes low well before a clock edge, the flip-flop does not immediately follow the input D.

For example, even though D goes high, the output Q waits until the next rising edge of the clock to transition to high.

This shows that Q is synchronized with the clock—although D can change anytime, Q only updates on the active clock edge.

__Asynchronous Reset Behavior:__

Consider the case when, during the previous rising clock edge, D = 1, so Q = 1.

The moment the reset is asserted (goes low), the output Q immediately drops to 0, without waiting for the next clock edge.

This confirms that the asynchronous reset overrides the clock and forces Q low instantly, independent of the clock signal.

Synthesis:
```bash
yosys
read_liberty -lib path/to/library.lib
read_verilog dff_asyncres.v
synth -top dff_asyncres
dfflibmap -liberty path/to/library.lib
abc -liberty path/to/library.lib
show
```

![DFF Async Reset Synth](images/dsynth_dff_asyncres.png)

---

### Example 2: D Flip-Flop with Asynchronous Set

```bash
iverilog dff_async_set.v tb_dff_async_set.v
./a.out
gtkwave tb_dff_async_set.vcd
```

![DFF Async Set Waveform](images/tb_dff_async_set.png)  
*From the waveform, we can observe the following behavior:*

__Normal Operation (without set):__
When the asynchronous set is inactive, the flip-flop behaves like a regular D flip-flop.

The output Q only updates on the rising edge of the clock and follows the value of D at that clock edge.

__Asynchronous Set Behavior:__

The moment the set signal is asserted (goes high), the output Q immediately becomes 1, regardless of the clock or the value of D.

While the set signal remains active, Q stays fixed at 1 and ignores any changes in D or the clock.

Once the set signal is deasserted (goes low), the flip-flop resumes normal operation, and Q again follows D only at the rising edges of the clock.*
Synthesis:
```bash
yosys
read_liberty -lib path/to/library.lib
read_verilog dff_async_set.v
synth -top dff_async_set
dfflibmap -liberty path/to/library.lib
abc -liberty path/to/library.lib
show
```

![DFF Async Set Netlist](images/synth_dff-async_set.png)


---

### Example 3: D Flip-Flop with Synchronous Reset

```bash
iverilog dff_syncres.v tb_dff_syncres.v
./a.out
gtkwave tb_dff_syncres.vcd
```

![DFF Sync Reset Waveform](images/tb_dff_syncres.png) 
 __Normal Operation (without reset):__

When the synchronous reset is inactive, the flip-flop behaves like a regular D flip-flop.

The output Q updates only on the rising edge of the clock and follows the value of D at that edge.

__Synchronous Reset Behavior:__

When the reset signal goes high, the output Q does not immediately go low (unlike asynchronous reset).

Instead, Q waits until the next rising edge of the clock to go low.

At that clock edge, the reset condition takes precedence over D, so the output is forced to 0.

After reset is deasserted (goes low), the flip-flop resumes normal operation, and Q once again follows D only at clock edges. 

Synthesis:
```bash
yosys
read_liberty -lib path/to/library.lib
read_verilog dff_syncres.v
synth -top dff_syncres
dfflibmap -liberty path/to/library.lib
abc -liberty path/to/library.lib
show
```

![DFF Sync Reset Netlist](images/synth_dff_syncres.png)

---

## ✅ Key Takeaways
- `.lib` files contain cell delay, power, and behavior info for synthesis.  
- Hierarchical synthesis preserves module structure; flat synthesis optimizes more.  
- Submodule synthesis improves scalability for repeated module designs.  
- Flip-flops are essential for stable circuits and come in different coding styles.  
- Practical synthesis and simulation steps were executed successfully using **Icarus Verilog, GTKWave, and Yosys**.  

---

