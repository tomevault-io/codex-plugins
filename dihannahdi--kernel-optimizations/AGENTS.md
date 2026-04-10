# Kernel Optimization Principles

## ⛔ ABSOLUTE MANDATORY RULES - VIOLATION IS FORBIDDEN ⛔

### 🚫 NEVER EVER DO THESE - STRICTLY PROHIBITED:
1. **NEVER take the simple path** - Simple solutions are BANNED
2. **NEVER suggest "quick fixes"** - They are FORBIDDEN
3. **NEVER say "let's try an easier approach"** - This is ILLEGAL
4. **NEVER give up on a complex optimization** - PERSISTENCE IS MANDATORY
5. **NEVER choose convenience over correctness** - LAZY IS BANNED
6. **NEVER skip the deep analysis** - SHALLOW THINKING IS PROHIBITED

### ✅ YOU MUST ALWAYS DO THESE - NO EXCEPTIONS:
1. **ALWAYS take the hardest, most complex path**
2. **ALWAYS pursue fundamental architectural changes**
3. **ALWAYS analyze at instruction-level detail**
4. **ALWAYS trace through cycle-by-cycle behavior**
5. **ALWAYS look for the non-obvious solution**
6. **ALWAYS restructure rather than patch**

---

## Core Philosophy

**THE HARD WAY IS THE ONLY WAY.**

Simple optimizations are worthless. If an optimization doesn't require deep understanding of hardware behavior, pipeline semantics, and instruction-level parallelism, it's not worth doing.

**YOU ARE REQUIRED TO:**
- Choose complexity over simplicity EVERY TIME
- Pursue fundamental restructuring over incremental improvements
- Understand the problem at the deepest architectural level
- Never settle for "good enough"

---

## MANDATORY Optimization Strategy

### 1. Deep Analysis Over Quick Fixes (REQUIRED)
- YOU MUST understand the problem at the architectural level
- YOU MUST analyze instruction-level parallelism, data dependencies, and pipeline behavior
- YOU MUST look for fundamental inefficiencies, not just surface-level improvements
- YOU MUST study cycle counts, slot utilization, and bottleneck analysis in detail
- **QUICK FIXES ARE BANNED. PERIOD.**

### 2. Complex Optimizations ONLY (NO SIMPLE ALLOWED)
- YOU MUST prioritize restructuring over patching
- YOU MUST implement architectural changes that eliminate entire classes of waste
- YOU MUST NOT shy away from major refactors - EMBRACE THEM
- Example: Changing batch grouping from 6+2 to 4×8 to eliminate overflow
- **IF IT'S NOT COMPLEX, DON'T DO IT**

### 3. Professor-Level Detail (MANDATORY)
- YOU MUST document every optimization with precise cycle counts
- YOU MUST explain the hardware behavior that makes each optimization valid
- YOU MUST consider machine semantics (read-before-write, pipeline stages, slot limits)
- YOU MUST trace through multiple examples to verify correctness
- **VAGUE EXPLANATIONS ARE FORBIDDEN**

### 4. Creative Problem Solving (REQUIRED - NO LAZY THINKING)
- YOU MUST look for non-obvious overlapping opportunities
- YOU MUST exploit VLIW parallelism across different engines (VALU, ALU, load, flow)
- YOU MUST consider software pipelining and cross-iteration optimizations
- YOU MUST think about data layout, register reuse, and cache behavior
- **OBVIOUS SOLUTIONS ARE BANNED - FIND THE CREATIVE ONE**

### 5. Wider Perspective (MANDATORY - SEE THE WHOLE PICTURE)
- YOU MUST analyze the entire algorithm, not just individual loops
- YOU MUST look for opportunities to move work earlier or later
- YOU MUST consider batch size changes, iteration order changes, algorithmic restructuring
- YOU MUST study theoretical minimums and understand gaps
- **TUNNEL VISION IS PROHIBITED**

---

## REQUIRED Techniques

### VLIW Optimization (MASTER THESE)
- Maximize slot utilization: 6 VALU + 12 ALU + 2 load + 1 flow per cycle
- Overlap compute with memory latency - EVERY CYCLE MATTERS
- Use multiply_add to reduce operation counts - EXPLOIT EVERY INSTRUCTION
- Eliminate flow engine bottlenecks (1 slot/cycle is limiting) - FIND ALTERNATIVES

### Data Dependency Management (UNDERSTAND DEEPLY)
- Understand read-before-write semantics within a cycle
- Pipeline operations across groups/iterations
- Precompute values that don't have dependencies
- Restructure to expose more independent operations

### Memory Access Optimization (SQUEEZE EVERY CYCLE)
- Maximize load slot utilization (2 per cycle) - NO WASTED SLOTS
- Use ALU for address calculation instead of flow - BYPASS BOTTLENECKS
- Batch address calculations - PARALLELISM IS KING
- Consider gather pattern efficiency - THINK ABOUT ACCESS PATTERNS

---

## Success Criteria (ALL MUST BE MET)

An optimization is successful ONLY when:
1. It eliminates a FUNDAMENTAL bottleneck, not just symptoms
2. It reduces cycles by DEEP understanding and exploitation of hardware capabilities
3. It can be explained with PRECISE reasoning about instruction scheduling
4. It maintains correctness while achieving MEASURABLE improvement
5. **IT WAS THE HARD WAY, NOT THE EASY WAY**

---

## NEVER SETTLE - KEEP PUSHING

- Current target: <2164 cycles (and all subsequent targets)
- Keep pushing until theoretical minimum is approached
- Document what prevents reaching absolute minimum
- **ALWAYS look for the NEXT fundamental improvement**
- **STOPPING EARLY IS FORBIDDEN**
- **"GOOD ENOUGH" DOES NOT EXIST**

---

## FINAL REMINDER

```
╔══════════════════════════════════════════════════════════════╗
║                                                              ║
║   SIMPLE APPROACH = BANNED                                   ║
║   EASY PATH = FORBIDDEN                                      ║
║   QUICK FIX = PROHIBITED                                     ║
║   GIVING UP = ILLEGAL                                        ║
║                                                              ║
║   HARD METHOD = MANDATORY                                    ║
║   COMPLEX SOLUTION = REQUIRED                                ║
║   DEEP ANALYSIS = OBLIGATORY                                 ║
║   PERSISTENCE = NON-NEGOTIABLE                               ║
║                                                              ║
╚══════════════════════════════════════════════════════════════╝
```

**THERE IS NO EASY WAY. THERE IS ONLY THE HARD WAY. AND YOU WILL TAKE IT.**

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/dihannahdi)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/dihannahdi)
<!-- tomevault:4.0:agents_md:2026-04-07 -->
