# Vieta's Poly
## Description

> Here's a pwntools tutorial challenge to get you warmed up!

> nc ctf.k3rn3l4rmy.com 2236

## Source

```py
from pwn import *
context.log_level = 'debug' #will print all input and output for debugging purposes
conn = remote("server",port) #enter the address and the port here as strings. For example nc 0.0.0.0 5000 turns into remote('0.0.0.0', 5000)
def get_input(): #function to get one line from the netcat
    input = conn.recvline().strip().decode()
    return input
def parse(polynomial):
    '''
    TODO: Parse polynomial
    For example, parse("x^3 + 2x^2 - x + 1") should return [1,2,-1,1]
    '''
for _ in range(4): get_input() #ignore challenge flavortext
for i in range(100):
    type = get_input()
    coeffs = parse(get_input())
    print(coeffs)
    ans = -1
    if 'sum of the roots' in type:
        ans = -1
        #TODO: Find answer
    elif 'sum of the reciprocals of the roots' in type:
        ans = -1
        #TODO: Find answer
    elif 'sum of the squares of the roots' in type:
        ans = -1
        #TODO: Find answer
    conn.sendline(str(ans)) #send answer to server
    get_input()
conn.interactive() #should print flag if you got everything right
```

## Solution

From the title of challenge, I thought it's relate to `vieta's formulas`. Yeah, I searched about this keyword in Google and copy a function to find `coeffs` 

And you know, it's failed to submit the flag. Cause the challenge intentaion is find the roots.

From source code:
- Parse equation to polynomial
- Calculate roots with given polynomial 
- Calculate `sum of sth` following the statement 

My code do the same thing with what I said

```py
from pwn import *
import re
import numpy as np
context.log_level='debug'

# Parse equation to polynomial
def parse(string):
    arr = re.split('\^\d+\s', string)
    newArr = []
    for x in arr:
        if x == 'x':
            newArr.append(1)
        else:
            newArr.append(int(x.replace(' ','').split('x')[0]))
    return newArr

def sumRoots(arr):
    return sum(arr)

def sumSquaresRoots(arr):
    sum = 0
    for i in arr:
        sum += i**2
    return sum

def sumReciprocalsRoots(arr):
    sum = 0 
    for i in arr:
        sum += 1/i
    return sum

# Check statement
def checkCondition(res):
    for i in range(len(res)):
        if "sum of the roots" in res[i]:
            print(["sum of the roots", res[i + 1]])
            return ["sum of the roots", res[i + 1]]
        elif "reciprocals" in res[i]:
            print(["reciprocals", res[i + 1]])
            return ["sum of the reciprocals of the roots", res[i + 1]]
        elif "squares" in res[i]:
            print(["sum of the squares roots", res[i + 1]])
            return ["sum of the squares of the roots", res[i + 1]]

# Receive response from the netcat
def get_input():
    input = p.recv().strip().decode()
    return input

# Find roots
def getRoots(polynomial):
    roots = np.roots(polynomial)
    return roots

p = remote('ctf.k3rn3l4rmy.com', 2236)

for i in range(100):
    res = get_input().split('\n')
    cmd = checkCondition(res)
    polynomial = parse(cmd[1].rstrip())
    roots = getRoots(polynomial)

    if 'sum of the roots' in cmd:
        ans = sumRoots(roots)
    elif 'sum of the reciprocals of the roots' in cmd:
        ans = sumReciprocalsRoots(roots)
    elif 'sum of the squares of the roots' in cmd:
        ans = sumSquaresRoots(roots)
    p.sendline(str(int(round(ans).real)))
    print("Sent", ans)

res = get_input()
p.close()
```

![Flag](./img/Vieta's%20Poly/flag.png)

The flag was captured 🎉🎉

```
Flag is : flag{Viet4s_f0r_th3_win}
```
