# BDump File Format

BDump v3 is a file format for storing Minecraft's structures. It is made of different commands that indicate the constructing process.

By writing the ids that represent each blocks in a specific order is a workable plan that reduces the file size, but this would allow a large amount of unexpected air blocks that increasing the file size so we implemented a new format which has a pointer that indicates where the "brush" is, the file is a set of commands that tells the "brush" how to move, and where to place blocks. Within this file format, air blocks can be simply skipped by a move command so files can be smaller.

## Basic File Structure

BDump v3 file's extension is `.bdx`, and the general header of it is `BD@`, which stands for that the file is compressed with brotli compression algorithm. Note that there's also a header `BDZ` that stands for the file is compressed with gzip compression algorithm, which is no longer supported by FastBuilder Phoenix today since it has been deprecated for a long time and it's hard to find this type's file again. We define this kind of header as "compression header"  and the content after it is compressed with the compression algorithm it indicates.

> Tip: BDump v2's extension is `.bdp` and the header is `BDMPS\0\x02\0`.

The header of the compressed content is `BDX\0`, and the author's player name that terminated with `\0` is followed right after it. Then the content after it is the command with arguments that written one-by-one tightly. Each command id would take 1 byte of space, like what an `unsigned char` do.

All the operations depend a `Vec3` value represents the current position of the "brush".

Let's see the list of commands first.

> Note: Integers would be written in <font style="color:red;">**big endian**</font>.
>
> What is the difference of little endian and big endian?
>
> For example, an int32 number in little endian, `1`, is `01 00 00 00` in the memory, and the memory of an int32 number `1` in big endian is `00 00 00 01`.

Type definition:

* {int}: a number that can be positive, negative or zero.
* {unsigned int}: a number that can be positive or zero.
* `char`: an {int} value with 1 byte long.
* `unsigned char`: an {unsigned int} value with 1 byte long.
* `short`: an {int} value with 2 bytes long.
* `unsigned short`: an {unsigned int} value with 2 bytes long.
* `int32_t`: an {int} value with 4 bytes long.
* `uint32_t`: an {unsigned int} value with 4 bytes long.
* `char *`: a string that terminated with `\0`.
* `int`: alias of `int32_t`
* `unsigned int`: alias of `uint32_t`
* `bool`: a value that can be either `true(1)` or `false(0)`, 1 byte long.

| ID                | Internal name               | Description                                                  | Arguments                                                    |
| ----------------- | --------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 1                 | `addToBlockPalette`         | Add a specific block name to the palette, and the id of the block is the times of using this command, e.g. the id of the first time calling the command is `0`, and the id of the second time is `1`. The maximum number of types of blocks is `65536`. | `char *blockName`                                            |
| 2                 | `addX`                      | **(DEPRECATED)** Add `x` to the brush position's `X`, and reset the value of `Y` and `Z` to `0`. This method is deprecated since the difference between the real function of the command and what it's name say it should do. Though it's deprecated, you still need to implement it's read since there's still bdx files contain this command. | `unsigned short x`                                           |
| 3                 | `X++`                       | **(DEPRECATED) **Add `1` to the brush position's `X`, and reset the value of `Y` and `Z` to `0`. | -                                                            |
| 4                 | `addY`                      | **(DEPRECATED)** Add `y` to the brush position's `Y`, and reset the value of `Z` to `0`. | `unsigned short y`                                           |
| 5                 | `Y++`                       | **(DEPRECATED) **Add `1` to the brush position's `Y`, and reset the value of `Z` to `0`. | -                                                            |
| 6                 | `addZ`                      | Add `z` to the brush position's `Z`, it's not deprecated since it resets nothing, though it's no longer used by the current version of PhonixBuilder. | `unsigned short z`                                           |
| 7                 | `placeBlock`                | Place a block on the current position of the brush with the id from the `addToBlockPalette` command and the `blockData` value. | `unsigned short blockID`<br/>`unsigned short blockData`      |
| 8                 | `Z++`                       | Add `1` to the brush position's `Z`, it's not deprecated since it resets nothing, though it's no longer used by the current version of PhonixBuilder. | -                                                            |
| 9                 | `NOP`                       | Do nothing. (No Operation)                                   | -                                                            |
| 10, `0x0A`        | `jumpX`                     | **(DEPRECATED)** Add `x` to the brush position's `X`, and reset the value of `Y` and `Z` to `0`. This method is deprecated since the difference between the real function of the command and what it's name say it should do. Though it's deprecated, you still need to implement it's read since there's still bdx files contain this command.<br/>The difference between `jumpX` and `addX` command is that `jumpX` uses `unsigned int` for its argument instead of `unsigned short`. | `unsigned int x`                                             |
| 11, `0x0B`        | `jumpY`                     | **(DEPRECATED)** Add `y` to the brush position's `Y`, and reset the value of `Z` to `0`. | `unsigned int y`                                             |
| 12, `0x0C`        | `jumpZ`                     | Add `z` to the brush position's `Z`, it's not deprecated since it resets nothing, though it's no longer used by the current version of PhonixBuilder. | `unsigned int z`                                             |
| 13, `0x0D`        | `reserved`                  | Reserved command, shouldn't be used by your program.         | ???                                                          |
| 14, `0x0F`        | `*X++`                      | Add `1` to the brush position's `X`.                         | -                                                            |
| 15, `0x10`        | `*X--`                      | Subtract `1` from the brush position's `X`.                  | -                                                            |
| 16, `0x11`        | `*Y++`                      | Add `1` to the brush position's `Y`.                         | -                                                            |
| 17, `0x12`        | `*Y--`                      | Subtract `1` from the brush position's `Y`.                  | -                                                            |
| 18, `0x13`        | `*Z++`                      | Add `1` to the brush position's `Z`.                         | -                                                            |
| 19, `0x14`        | `*Z--`                      | Subtract `1` from the brush position's `Z`.                  | -                                                            |
| 20, `0x15`        | `*addX`                     | Add `x` to the brush position's `X`. `x` could be either positive, negative or zero. | `short x`                                                    |
| 21, `0x16`        | `*addBigX`                  | Add `x` to the brush position's `X`. The difference between this command and the previous one is this command uses `int32` as its argument. | `int x`                                                      |
| 22, `0x17`        | `*addY`                     | Add `y` to the brush position's `Y`.                         | `short y`                                                    |
| 23, `0x18`        | `*addBigY`                  | Add `y` to the brush position's `Y`.                         | `int y`                                                      |
| 24, `0x19`        | `*addZ`                     | Add `z` to the brush position's `Z`.                         | `short z`                                                    |
| 25, `0x1A`        | `*addBigZ`                  | Add `z` to the brush position's `Z`.                         | `int z`                                                      |
| 26, `0x1B`        | `assignCommandBlockData`    | Set the command block data for the block at the brush's position. | `unsigned int mode {Impulse=0, Repeat=1, Chain=2}`<br/>`char *command`<br/>`char *customName`<br/>`char *lastOutput`<br/>`int tickdelay`<br/>`bool executeOnFirstTick`<br/>`bool trackOutput`<br/>`bool conditional`<br/>`bool needRedstone` |
| 27, `0x1C`        | `placeCommandBlockWithData` | Place a command block, and set its data at the brush's position. | `unsigned short blockID`<br/>`unsigned short blockData`<br/>`unsigned int mode {Impulse=0, Repeat=1, Chain=2}`<br/>`char *command`<br/>`char *customName`<br/>`char *lastOutput`<br/>`int tickdelay`<br/>`bool executeOnFirstTick`<br/>`bool trackOutput`<br/>`bool conditional`<br/>`bool needRedstone` |
| 28, `0x1D`        | `addSmallX`                 | Add `x` to the brush position's `X`. The difference between this command and the `*addX` command is that this command uses `char` as its argument. | `char x //int8_t x`                                          |
| 29, `0x1E`        | `addSmallY`                 | Add `y` to the brush position's `Y`.                         | `char y //int8_t y`                                          |
| 30, `0x1F`        | `addSmallZ`                 | Add `z` to the brush position's `Z`.                         | `char z //int8_t z`                                          |
| 88, `'X'`, `0x58` | `end`                       | Stop reading. Note that though the general end is "XE" (2 bytes long), but a 'X' (1 byte long) character is enough. | -                                                            |

The list above is all the commands of the bdump v3 till 2021-8-14.

Let's see how to make a `bdx` file using these commands.

If we want to place a TNT block at `{3,5,6}`(**relative**), and a repeating command block with command `kill @e[type=tnt]` and name `Kill TNT!` that doesn't need redstone to be activated at `{3,6,6}`, then a glass block at `{114514,15,1919810}` and a iron block at `{114514,15,1919800}`, the uncompressed bdx file might be:

`BDX\0DEMO\0\x01tnt\0\x1D\x03\x01repeating_command_block\0\x01glass\0\x01iron_block\0\x1F\x06\x1E\x05\x07\0\0\0\0\x11\x1C\0\x01\0\0\x01kill @e[type=tnt]\0Kill TNT!\0\0\0\0\0\0\x01\x01\0\0\x1E\x09\x1A\0\x1D\x4B\x3C\x16\0\x01\xBF\x4F\x07\0\x02\0\0\x1F\xF6\x07\0\x03\0\0XE`

The pseudo assembly code form of this file is:

```assembly
author 'DEMO\0'
addToBlockPalette 'tnt\0' ; ID: 0
addSmallX 3 ; brushPosition: {3,0,0}
addToBlockPalette 'repeating_command_block\0' ; ID: 1
addToBlockPalette 'glass\0' ; ID: 2
addToBlockPalette 'iron_block\0' ; ID: 3
addSmallZ 6 ; brushPosition: {3,0,6}
addSmallY 5 ; brushPosition: {3,5,6}
placeBlock (int16_t)0, (int16_t)0 ; TNT Block will be put at {3,5,6}
NewYadd ; *Y++, brushPosition: {3,6,6}
placeCommandBlockWithData (int16_t)1, (int16_t)0, 1, 'kill @e[type=tnt]\0', 'Kill TNT!\0', '\0', (int32_t)0, 1, 1, 0, 0 ; A command block will be put at {3,6,6}
addSmallY 9 ; brushPosition: {3,15,6}
addBigZ 1919804 ; 1919810: 00 1D 4B 3C = 01d4b3ch, brushPosition: {3,15,1919810}
addBigX 114511 ; 114511: 00 01 BF 4F = 01bf4fh, brushPosition: {114514,15,1919810}
placeBlock (int16_t)2,(int16_t)0 ; A glass block will be put at {114514,15,1919810}
addSmallZ -10 ; -10: F6 = 0f6h, brushPosition: {114514,15,1919800}
placeBlock (int16_t)3,(int16_t)0 ; A iron block will be put at {114514,15,1919800}
end
db 'E'
```
