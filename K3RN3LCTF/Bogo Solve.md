# Bogo Solve
## Description

> I was trying to sort using Bogosort, but it was taking a while, so in the meantime I made this challenge. Enjoy!

> nc ctf.k3rn3l4rmy.com 2237

## Source

```py
import random
NUMS = list(range(10**4))
random.shuffle(NUMS)
tries = 15
while True:
    try:
        n = int(input('Enter (1) to steal and (2) to guess: '))
        if n == 1:
            if tries==0:
                print('You ran out of tries. Bye!')
                break
            l = map(int,input('Enter numbers to steal: ').split(' '))
            output = []
            for i in l:
                assert 0<= i < len(NUMS)
                output.append(NUMS[i])
            random.shuffle(output)
            print('Stolen:',output)
        elif n == 2:
            l = list(map(int,input('What is the list: ').split(' ')))
            if l == NUMS:
                print(open('flag.txt','r').read())
                break
            else:
                print('NOPE')
        else:
            print('Not a choice.')
    except:
        print('Error. Nice Try...')
```

## Solution

Analyst the source code:
- Server holds a list having `10**4` random elements
- To capture the flag, we have to send exactly the list
- Request to steal numbers with your expected indexes
- Response from server for stolen numbers is random

Well, a naive approach is request 1 number each request

--> The connection will be closed soon if you request a lot

`WHAT IF` you send a list number like this `0 1 1 2 2 2 3 3 3` ? Yeah, the response will be `10 11 11 12 12 13 13 13` (`obviously it's random :D`)

Now, mapping from request to response, you'll know the index of `10` is `0` and so on

With this theory, I request `a certain number` and mapping it to response to create a list (payload in the future)

My code:
- Use `dictionary` to check and append my list
- I request `400 numbers` in a request (if this amount is too large, the connection will be closed)
- When my list is filled out, I'll send it to capture the flag

```py
from pwn import *
from collections import Counter
# context.log_level='debug'

def createListNum(begin, offset = 400):
    finalList = []
    for currentNum in range(begin, begin + offset):
        for num in range(begin, currentNum + 1):
            finalList.append(str(num))
    return ' '.join(finalList)
    # return finalList

def getKey(dict, value):
    return list(dict.keys())[list(dict.values()).index(value)]

p = remote('ctf.k3rn3l4rmy.com', 2237)
# Payload will be sent in the final request
REALNUMS = []
begin = 0

while begin != 10000:
    p.recvuntil('Enter (1) to steal and (2) to guess')
    # Send command 1
    p.sendline(bytes('1', 'utf-8'))
    p.recvuntil('Enter numbers to steal')
    print("Sent from", begin)
    # Send request
    request = createListNum(begin)
    p.sendline(request)

    # Receive response
    # Example response: Stolen: [5219, 1111, ..., 1]
    numList = p.recvline().strip().decode('utf-8')
    print("Received", numList)
    numList = numList.split('[')[1][:-1].split(', ')
    # Convert to dict
    response = Counter(numList)


    # Put into our payload
    for currentNum in range(begin, begin + 400):
        key = currentNum % 400
        REALNUMS.append(getKey(response, 400 - key))

    print("Len of received numbers", len(REALNUMS))
    # New request
    begin += 400
    if len(REALNUMS) == 10**4:
        break

p.recvuntil('Enter (1) to steal and (2) to guess')
# Send command 2
p.sendline(bytes('2', 'utf-8'))
p.recvuntil('What is the list')
print("Sent REALNUMS")
p.sendline(' '.join(REALNUMS))

# Receive flag
print(p.recvline().strip().decode('utf-8'))

p.close()
```

![Flag](./img/Bogo%20Solve/flag.png)

The flag was captured 🎉🎉

```
Flag is : flag{alg0r1thms_ar3_s0_c00l!}
```