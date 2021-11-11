# A Chest Full Of Cookies
## Description
> Only admin knows where he hides the flag. Connect at chall.ctf-ehcon.ml:32102
## Solution
After register and login to the application. I had a cookie name `isAdmin` with value based `base64`
Decode the value and get `false`. Okay it's quite clear, I base64 encode `true` and change my cookie. Reload the page and got nothing? Wut?
After a few times, I used `curl` to finish this challenge

```
Flag is : EHACON{y0u_@r3_@dm1n_n0w}
```