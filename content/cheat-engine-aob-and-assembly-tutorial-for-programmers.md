---
title: Cheat Engine AOB and Assembly Tutorial for Programmers
draft: false
tags:
  - fun
date: 2025-08-22
---
## Prerequisites

- **Java/Python Background**: You understand programming concepts like variables, functions, memory allocation
- **Single Player Games Only**: This tutorial is for educational purposes on single player games only
- **Cheat Engine Installed**: Download from [cheatengine.org](https://cheatengine.org/)

## Table of Contents

1. [Understanding Assembly Language](#understanding-assembly-language)
2. [What are AOBs (Array of Bytes)](#what-are-aobs-array-of-bytes)
3. [Basic Assembly Instructions](#basic-assembly-instructions)
4. [Finding Values and Memory Addresses](#finding-values-and-memory-addresses)
5. [Creating Your First AOB Script](#creating-your-first-aob-script)
6. [Auto Assembler and Code Injection](#auto-assembler-and-code-injection)
7. [Practical Examples](#practical-examples)
8. [Advanced Techniques](#advanced-techniques)
9. [Troubleshooting and Best Practices](#troubleshooting-and-best-practices)

---

## Understanding Assembly Language

### Assembly vs High-Level Languages

Coming from Java/Python, think of assembly as the lowest level of programming before machine code:

```
High Level (Java):    int health = 100;
Assembly:            mov [ebx+480], eax    ; Move value in EAX to memory location
Machine Code:        89 83 80 04 00 00    ; Raw bytes the CPU understands
```

### CPU Registers (Think of them as super-fast variables)

- **EAX, EBX, ECX, EDX**: General purpose registers (like temporary variables)
- **ESI, EDI**: Source/Destination index registers
- **ESP**: Stack pointer (points to current stack location)
- **EBP**: Base pointer (frame reference for function calls)
- **EIP**: Instruction pointer (points to next instruction to execute)

**32-bit vs 64-bit naming:**

- 32-bit: EAX, EBX, ECX, etc.
- 64-bit: RAX, RBX, RCX, etc. (R prefix)

---

## What are AOBs (Array of Bytes)?

### The Problem AOBs Solve

In Java, you might access a variable by name. In assembly, you access memory by address. But addresses change when:

- Game updates
- Different game launches
- Different computers

### AOB as a Solution

An AOB is just an Array of Bytes, it tends to be used as a signature. A signature is really only an AOB with wild cards. A signature can be found even if the address where it is changes, so long as the signature still exists.

**Example:**

```
Static Address (fragile): Tutorial-i386.exe+26188
AOB Pattern (flexible):   89 42 18 8B 45 DC 8B 40 18
AOB with wildcards:       89 ?? ?? 8B ?? ?? 8B ?? ??
```

**Benefits of AOBs:**

- They are dynamic and don't rely on static memory addresses
- They remain effective across game updates (as long as the core logic remains unchanged)
- They enable direct control over game logic

---

## Basic Assembly Instructions

### Data Movement Instructions

```assembly
mov eax, 100          ; Move immediate value 100 into EAX register
mov eax, [ebx]        ; Move value at memory address EBX into EAX
mov [ebx], eax        ; Move value in EAX to memory address EBX
lea eax, [ebx+4]      ; Load Effective Address (like &variable in C)
```

### Arithmetic Instructions

```assembly
add eax, 10           ; EAX = EAX + 10
sub eax, 5            ; EAX = EAX - 5
inc eax               ; EAX = EAX + 1
dec eax               ; EAX = EAX - 1
mul ebx               ; EAX = EAX * EBX
```

### Stack Operations (Function calls)

```assembly
push eax              ; Save EAX value on stack
pop ebx               ; Restore value from stack to EBX
call 0x12345678       ; Call function at address (like method call)
ret                   ; Return from function
```

### Control Flow

```assembly
cmp eax, 100          ; Compare EAX with 100
je label              ; Jump if Equal (like if statement)
jne label             ; Jump if Not Equal
jmp label             ; Unconditional jump (like goto)
```

---

## Finding Values and Memory Addresses

### Step 1: Basic Value Scanning

1. **Launch your single-player game**
2. **Open Cheat Engine and attach to game process**
3. **Find a value to modify** (health, ammo, money)

```
Example with health = 100:
1. Scan for exact value: 100
2. Take damage in game (health = 75)
3. Next scan for: 75
4. Repeat until you find the address
```

### Step 2: Finding What Writes to Address

Right-click the memory address and select "Find out what writes to this address."

This shows you the assembly instruction that modifies your value:

```assembly
mov [esi+480], eax    ; This instruction writes to your health
```

### Step 3: Analyze the Assembly Code

Right-click the instruction → "Show Disassembler" to see surrounding code:

```assembly
004523A0 - push ebp
004523A1 - mov ebp, esp
004523A3 - mov eax, [ebp+8]
004523A6 - mov [esi+480], eax    ; <-- Our target instruction
004523AC - pop ebp
004523AD - ret
```

---

## Creating Your First AOB Script

### Step 1: Extract the AOB Pattern

Right-click the instruction and select "Copy bytes" → "Copy bytes and Opcodes".

Example result:

```
89 86 80 04 00 00 - mov [esi+00000480],eax
```

### Step 2: Create Wildcards for Flexibility

Replace dynamic addresses with ?? wildcards to maintain compatibility across updates.

```
Original:     89 86 80 04 00 00
With wildcards: 89 86 ?? ?? ?? ??
```

### Step 3: Build the AOB Script

Open Auto Assembler (Ctrl+Alt+A) and create:

```assembly
[ENABLE]
// Find our pattern in memory
aobscan(healthHook, 89 86 ?? ?? ?? ??)
registersymbol(healthHook)

// Modify the instruction
healthHook:
mov [esi+00000480], #9999  ; Set health to 9999 instead

[DISABLE]
// Restore original instruction
healthHook:
mov [esi+00000480], eax    ; Original instruction
unregistersymbol(healthHook)
```

---

## Auto Assembler and Code Injection

### Understanding Auto Assembler

Cheat Engine uses it's own assembler it's called "Auto Assembler", but it understands more then just assembly

**Auto Assembler Features:**

- Assembly instructions
- Memory allocation commands
- Symbol registration
- Template generation

### Code Injection Template

For more complex modifications, use code injection:

```assembly
[ENABLE]
aobscan(healthHook, 89 86 80 04 00 00)
alloc(newmem, 1024)        ; Allocate memory for our code
registersymbol(healthHook)

healthHook:
jmp newmem                 ; Jump to our custom code
nop                        ; Fill remaining bytes with NOPs

newmem:
push eax                   ; Preserve registers
cmp eax, 0                 ; Check if damage would make health <= 0
jle setMinHealth           ; If so, set minimum health
mov [esi+00000480], eax    ; Otherwise, normal damage
jmp returnPoint

setMinHealth:
mov eax, 1                 ; Set health to 1 (never die)
mov [esi+00000480], eax

returnPoint:
pop eax                    ; Restore registers
jmp healthHook+6           ; Return to game code

[DISABLE]
healthHook:
mov [esi+00000480], eax    ; Restore original
unregistersymbol(healthHook)
dealloc(newmem)
```

---

## Practical Examples

### Example 1: Infinite Ammo

```assembly
[ENABLE]
aobscan(ammoHook, 29 87 ?? ?? ?? ??)  ; sub [edi+offset], eax
registersymbol(ammoHook)

ammoHook:
// Replace subtraction with addition instead
add [edi+000004A0], eax

[DISABLE]
ammoHook:
sub [edi+000004A0], eax    ; Original: subtract ammo
unregistersymbol(ammoHook)
```

### Example 2: Speed Hack

```assembly
[ENABLE]
aobscan(speedHook, F3 0F 11 ?? ?? ?? ?? ??)  ; movss instruction
alloc(newmem, 1024)
registersymbol(speedHook)

speedHook:
jmp newmem

newmem:
push eax
mov eax, (float)2.0        ; 2x speed multiplier
movss [ebx+000001C0], xmm0
mulss xmm0, [eax]          ; Multiply speed by 2
pop eax
jmp speedHook+8

[DISABLE]
speedHook:
movss [ebx+000001C0], xmm0 ; Original instruction
unregistersymbol(speedHook)
dealloc(newmem)
```

---

## Advanced Techniques

### Working with Floating Point Values

```assembly
; For floating point operations
movss xmm0, [ebx+4]        ; Move single precision float
addss xmm0, xmm1           ; Add floats
mulss xmm0, [speedMultiplier] ; Multiply by stored value
```

### Multi-Level Pointers

When values are stored through multiple pointer levels:

```assembly
mov eax, [playerBase]      ; Get player object
mov eax, [eax+10]          ; Get health object
mov [eax+4], 9999          ; Set health value
```

### Conditional Modifications

```assembly
cmp [playerID], 1          ; Check if it's the player
jne skipModification       ; Skip if not player
mov [eax+480], 9999        ; Apply god mode to player only
skipModification:
```

---

## Troubleshooting and Best Practices

### Common Mistakes

1. **Not preserving registers**: Always use `push`/`pop` to save register values
2. **Wrong jump distances**: Ensure your jumps don't exceed 127 bytes for short jumps
3. **Forgetting to restore original code**: Always implement proper `[DISABLE]` sections

### Memory Safety

Always create a backup before modifying memory.

```assembly
; Good practice: preserve all registers
pushad                     ; Push all general registers
pushfd                     ; Push flags register
; ... your code here ...
popfd                      ; Restore flags
popad                      ; Restore all registers
```

### Debugging Tips

1. **Use breakpoints**: Right-click instruction → "Set breakpoint"
2. **Check register values**: Use debugger to inspect register contents
3. **Test incrementally**: Start with simple modifications before complex logic
4. **Use NOPs for testing**: Replace instructions with NOPs to test if they're crucial

### Script Structure Best Practices

```assembly
[ENABLE]
aobscan(hookName, byte_pattern)     ; Find pattern
alloc(newmem, 1024)                 ; Allocate memory
registersymbol(hookName)            ; Register for table

hookName:
jmp newmem                          ; Jump to our code
nop                                 ; Fill remaining bytes

newmem:
; Your modification code here
jmp hookName+original_instruction_size

[DISABLE]
hookName:
db original_bytes                   ; Restore original bytes
unregistersymbol(hookName)
dealloc(newmem)
```

---

## Learning Resources

### Official Documentation

- **Cheat Engine Wiki**: [wiki.cheatengine.org](https://wiki.cheatengine.org)
- **Auto Assembler Commands**: Complete reference for all CE commands
- **Assembly Reference**: x86 instruction set documentation

### Practice Recommendations

1. **Start with CE Tutorial**: Complete the built-in Cheat Engine tutorial
2. **Use Simple Games**: Practice on older/simpler games first
3. **Join Communities**: Cheat Engine forums and FearLess Revolution
4. **Study Existing Tables**: Download and analyze community cheat tables

### Key Concepts to Master

- Register preservation and stack management
- Understanding memory layouts and pointer chains
- AOB pattern creation with appropriate wildcards
- Code injection vs direct memory modification
- Debugging and troubleshooting techniques