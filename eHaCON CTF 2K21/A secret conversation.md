# A secret conversation
## Description
> DSG 1342 : Dark Gray knows where you kept the secret
> OEH 4875 : No, I did not hide anything. Ask RFC.
> RFC 4287 : Mark Nottingham forgets everything.
> RFC 2010 : I am always right.
> Connect the server at chall.ctf-ehcon.ml:32103
## Solution
Actually, I always brute force with challengs just has a single page as this challenge. I got an endpoint `/cgi/test-cgi`, after a lot of times try to find vulnerability in this endpoint, I decied to bought hint 😀 and I got 
```
Mark has achieved somthing in the year 2010
```
Search about `Mark Nottingham` in 2010


We saw something related to `RFC` right? It's  `RFC 5988` in 2010. 


Click it to view more detail and find with key `2010`. I got 2 papers was released in 2010

The next step is search about 2 RFC standard in CTF
- `RFC 5988`
- `RFC 5785`
I tried with `5988` first, when it was failed I step to `5785` and got a endpoint 😀 `.well-known` 

Access the file `flag.txt` to capture the flag
```
Flag is : EHACON{rfc_fr0m_13tf} 
```