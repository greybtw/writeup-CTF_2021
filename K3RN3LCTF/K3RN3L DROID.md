# K3RN3L DROID
## Description

> I have build my own system that allow to log-in into K3RN3L DR0ID, but unfortunately i forgot the password, can you help me recover it?

## Source

I attached in `../src/kernel-droid.asm` :D Check it out if you want

## Solution

The first, find the length of PIN
- This label call `invalid_pin_length` 
- Condition to jump to `invalid_pin_length` is `jne` (`jump not equal`)

--> Length is `0x8 = 8`
```as
label4:
    lea rdi, [pinCode]
    call gets
    mov rdi, rax
    call strlen
    cmp rax, 0x8
    jne invalid_pin_length 
    je label5
    jmp end
``` 

Next, identify each character in PIN in `label5` (this is snippet code)
- The trick is reverse the conditioon ψ(｀∇´)ψ

For example the first char. We will reverse the condition in comparation to eliminate the invalid value range. Therefore, `jne invalid_pin => if PIN + 0x0 != 0x30, jump invalid_pin`, and result is `PIN + 0x0 = 0x30 --> don't jump ❌`
- `equal 0x30`
```py
    mov r9b, [pinCode + 0x0]
    cmp r9b, 0x30
    jne invalid_pin
    cmp r9b, 0x31
    je invalid_pin
    cmp r9b, 0x32
    je invalid_pin
    cmp r9b, 0x35
    je invalid_pin
```
The second
- `equal 0x34` 
```py
    mov r9b, [pinCode + 0x1]
    cmp r9b, 0x34
    jne invalid_pin
```
The third
- `less or equal than 0x37` 
- `equal 0x30`
```py
    mov r9b, [pinCode + 0x2]
    cmp r9b, 0x37
    jg invalid_pin
    cmp r9b, 0x30
    jne invalid_pin
```

4
- `equal 0x30` 
```py
    mov r9b, [pinCode + 0x3]
    cmp r9b, 0x39
    je invalid_pin
    cmp r9b, 0x30
    jne invalid_pin
```

5
- `less or equal 0x31` 
- `not below 0x31`
```py
    mov r8b, [pinCode + 0x4]
    cmp r8b, 0x31
    jb invalid_pin
    cmp r8b, 0x32
    jge invalid_pin
```

6
- `not below or equal 0x30`
- `not above or equal 0x32`
```py
    mov r8b, [pinCode + 0x5]
    cmp r8b, 0x30
    jbe invalid_pin
    cmp r8b, 0x32
    jae invalid_pin
```
7
- `equal 0x39`
```py
    mov r8b, [pinCode + 0x6]
    cmp r8b, 0x39
    jne invalid_pin
```
8
- `greater or equal 0x36`
- `less or equal 0x36`
```py
    mov r8b, [pinCode + 0x7]
    cmp r8b, 0x36
    jl invalid_pin
    cmp r8b, 0x36
    jg invalid_pin
    jmp valid_pin
```


Summary: `AND` each condition in each label, you'll have

```py
length = 8
[pinCode + 0x0] = 0x30
[pinCode + 0x1] = 0x34
[pinCode + 0x2] = 0x30
[pinCode + 0x3] = 0x30
[pinCode + 0x4] = 0x31
[pinCode + 0x5] = 0x31
[pinCode + 0x6] = 0x39
[pinCode + 0x7] = 0x36
--> Convert to ascii 04001196
```

In section `.data` define form of flag
```as
section .data
    welcomeMessage db "Welcome in KernelDroid, please input your PIN code to enter into phone", 0
    invalidPinLengthMessage db "I'm sorry your pin is not valid, at least it should be 8 (0-9) numbers length for example. 123456789", 0
    invalidPinMessage db "This is not valid PIN!", 0
    validPinMessage db "Great, your flag flag{K3RN3L_DR0ID_%s}", 0
```

Replace our PIN into flag and the flag was captured 🎉🎉

``` 
Flag is : flag{K3RN3L_DR0ID_04001196}
```