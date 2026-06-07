# Crypto — Custom CPU Emulator

**CTF:** PSRU Cyber Hackathon #3  
**Category:** Cryptography / Reverse Engineering  
**Flag:** `C0D3_0N_F1R3`

## Description

> We extracted a `rom.asm` file from a hacker's C&C server. It turned out to be instructions for a custom rogue CPU architecture the attacker built themselves to hide secret data. Your job is to write an emulator to run all these instructions.
>
> **Architecture spec:**
> - 4 registers: R1, R2, R3, R4 (all initialized to 0)
> - Memory: 256 slots (addresses 0–255, all initialized to 0)
> - Memory wrap rule: if an instruction references an address above 255, wrap around (e.g. address 256 = slot 0, address 260 = slot 4)

## Solution

The challenge gave us a `rom.asm` file containing instructions for a made-up CPU architecture. The goal was to simulate that CPU and recover whatever it computed.

### Step 1 — Understand the architecture

The spec defined a minimal CPU:
- 4 general-purpose registers (R1–R4), all starting at 0
- 256-byte memory, all starting at 0
- Memory accesses wrap at 256 (modulo arithmetic)

### Step 2 — Write the emulator

I wrote a Python emulator that parsed each instruction in `rom.asm` and executed it faithfully, implementing the memory wrap rule with `address % 256`.

```python
registers = {'R1': 0, 'R2': 0, 'R3': 0, 'R4': 0}
memory = [0] * 256

# ... parse and execute each instruction from rom.asm
# memory wrap example:
def mem_read(addr):
    return memory[addr % 256]

def mem_write(addr, val):
    memory[addr % 256] = val
```

### Step 3 — Spot the XOR decryption loop

After running the emulator, I noticed the program was writing encoded values into memory and then looping over them, XORing each byte with the key `73`.

```python
flag_bytes = [memory[i] for i in range(some_start, some_end)]
flag = ''.join(chr(b ^ 73) for b in flag_bytes)
print(flag)
```

### Step 4 — Get the flag

Running the emulator produced the flag directly as output.

## Flag

`C0D3_0N_F1R3`

## Takeaways

- Custom CPU emulation is a classic RE technique — always read the architecture spec carefully before writing code
- When you see a loop iterating over memory bytes with a fixed operation, suspect XOR encryption with a static key
- Bruteforcing the XOR key (0–255) is trivial if you don't spot it immediately — just check which key produces readable ASCII
