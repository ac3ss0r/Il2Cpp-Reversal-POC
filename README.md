# Il2Cpp Reversal POC (Proof-Of-Concept)
### Theory
Il2cpp is a technology used for converting from [CIL](https://ru.wikipedia.org/wiki/Common_Intermediate_Language) to machine code. It is used in Unity game engine for game optimization and most popular Unity games, for example Pixel Gun 3d.
<p align="center"><img src="https://github.com/ac3ss0r/Il2Cpp-Reversal-POC/blob/main/il2cpp.png?raw=true"></p>
There is no way to fully convert Il2cpp binaries back to CIL so for cracking those games and applications we need to patch the native assembly code itself. 

There are 2 main methods of assembly patching:

- By byte (hex) signatures - in this method the algorithm searches in memory the specified signature and replaces it with new bytes. The advantage of this method is that the method signatures often stay the same for a while (so the patch will be working fine even in different game versions). The disadvantage is that method signatures sometimes match, causing you to patch not only the method you wanted to, but other ones too (and possibly crash the application).

- By offsets - in his method the algorithm searches a specified offset in memory (from the start of specified .so library) and replaces the bytes, starting at that offset. This method is more efficient and stable in many cases, but the disadvantage is that the offsets almost always change in different game versions, and you'll need to find them over and over so the patch works in newer updates.

### Practice
I've created a test application made in unity with Il2cpp to show in can be used.

<p align="center"><img src="https://github.com/ac3ss0r/Il2Cpp-Reversal-POC/blob/main/Poc1.jpg?raw=true"></p>

And made a lua Game Guardian library that supports patching using both HEX signature and offset methods, it can be found in <a href="https://github.com/ac3ss0r/Il2Cpp-Reversal-POC/blob/main/patchLib.lua">patchLib.lua</a>. So now we can start exploiting.

First I dump the assemblies using <a href="https://github.com/Perfare/Il2CppDumper">Il2CppDumper</a>, this gives us every method signature

<p align="center"><img src="https://github.com/ac3ss0r/Il2Cpp-Reversal-POC/blob/main/dnspy.png?raw=true"></p>

The bool in the application itself always returns true. Lets change the return value to false, for example.

```lua
require('patchLib') -- we import the library

gg.setRanges(gg.REGION_CODE_APP | gg.REGION_C_DATA) -- set up memory regions 
printOffset("libil2cpp.so", 0x524EA4, 8) -- we print out the bytes before the patch
patchOffset("libil2cpp.so", 0x524EA4, "00 00 A0 E3 1E FF 2F E1") -- patch by offset
printOffset("libil2cpp.so", 0x524EA4, 8) -- print bytes after the patch
```

The "00 00 A0 E3 1E FF 2F E1" is basically 

``` asm
mov r0, #0
bx lr
```

in arm assembly which means return 0 or  return false.

Now we execute the script using game guardian and press "Get values". And it works as expected - the method now returns false always and we see that in output.

<p align="center"><img src="https://github.com/ac3ss0r/Il2Cpp-Reversal-POC/blob/main/Poc2.jpg?raw=true"></p>

We can patch each method in the same way we did with the boolean one. All the examples used in this project are avaliable in this repository.
