# split

This challenge has the same basic elements as the previous with some minor changes, but here it is written out:

From reading the information on the website and running the program, we have the following information:
- The program will read 52 bytes of data into a 32 byte buffer
- There is a `usefulFunction` function that will put `usefulString` (containing `/bin/ls`) into `edi` before calling `system()`. In other words, putting `/bin/ls` into `edi` before calling `system()` creates the call: `system('/bin/ls`')`, which will print the contents of the current directory. We cannot just jump to this function since it will just execute `ls`, so we instead want to use a ROP gadget to change `edi` before calling `system()`. Conviniently, there is a `usefulString` in the binary that contains `/bin/cat flag.txt`, which is what we want to execute to print the flag.

Note that I found this information pretty quickly using `nm split` to print out all of the symbols (which informed me about `usefulFunction` and `usefulString`). I read the assembly to determine what was going on in that function in gdb via `disass usefulFunction`.

Other relevant information found via `checksec` is (which is the same as the previous challenge):
- There are no canaries
- PIE is not enabled
- NX is enabled

With this information, my approach is to:
1. Determine the offset of the return address on the stack (which is the same as the last challenge, so this was skipped)
2. Replace it with the address of a gadget to put `/bin/cat flag.txt` into `edi`
3. Add a call to `system()` to then execute our command

Since I've completed these before by manually finding gadgets, I decided to explore using the ROP functionality in pwntools. This made it very simple to create a rop chain, and is a follows:

```python
rop = ROP(exe) # create the rop object

rop.call(rop.ret) # ret to fix stack alignment issues
rop.system(exe.sym['usefulString']) # single line that creates a call to system and puts /bin/cat flag.txt into edi

payload = b'A' * 40
payload += rop.chain()

io.sendline(payload)
```
