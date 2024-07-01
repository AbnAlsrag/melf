This is the specification of the executable format called MELF (Minimal Executable and Linkable Format) for executables, dlls, object files and core dumps.

# TODO

- [x] **Relocation**
- [x] **Dynamic linking**
- [ ] **Compression**
- [ ] **Encryption**
- [ ] **Digital Signature**
- [x] **Destructor**
- [ ] **Optimize the spec size wise (especially relocation entries)**
- [ ] **Core Dumps**
- [ ] **Change dynamic loading to not be dependent on the module being a file**
- [ ] **Make the format more generic and not dependent on ascii**
- [ ] **Create a debugging format**

--------------------------------------------------------------------------

# Specification Version
The current version of this specification document is: 0

# General Concepts

| Name    | Description                                                                                                                  |
| ------- | ---------------------------------------------------------------------------------------------------------------------------- |
| Segment | The basic block making the MELF format these are hardcoded in the format, every field in the [[#Overview]] is a segment.     |
| Section | The basic building block of the program image it specifies where each part of the file should be loaded in memory if needed. |

# Overview

| Field            | Description                                                                                                |
| ---------------- | ---------------------------------------------------------------------------------------------------------- |
| header           | The header of the binary, for more info see [[MELF Format#Header\|Header]].                                |
| string table     | A table containing all the strings needed for the module.                                                  |
| section table    | A table of all the sections in the binary, for more info see [[MELF Format#Section Entry\|Section Entry]]. |
| symbol table     | A table of all the symbols in the binary, for more info see [[MELF Format#Symbol Entry\|Symbol Entry]].    |
| import table     | A table of all the needed MELF modules to load, for more info see [[#Import Entry]].                       |
| relocation table | A table of all the relocation entries needed for relocation, for more info see [[#Relocation Entry]].      |
| data             | The rest of the data of the binary.                                                                        |

## Platform Specifier Format
The platform specifier format is the format used for specifying the platforms that the binary supports, each platform name is separated with the ascii unit separator 0x1f, if the string is null then the section or the module has no platform and shouldn't be loaded.

## Header

| Offset | Size | Field             | Description                                                                                                                                                                                     |
| ------ | ---- | ----------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 0      | 8    | magic             | A string of value "\x00\x5DMELF\x5D\x00" is the identifier of a MELF module and should always be in this value.                                                                                 |
| 8      | 8    | version           | The melf specification version number, to know the current specification version see [[#Specification Version]].                                                                                |
| 16     | 8    | time stamp        | The time of creation of this module in Unix time.                                                                                                                                               |
| 24     | 8    | check sum         | It is the sum of all the bytes of the binary including the header but with the check sum set to 0x00.                                                                                           |
| 32     | 8    | platform          | A string index into the string table that specifies the platforms that this binary supports using the platform specifier format, for more about this format see [[#Platform Specifier Format]]. |
| 40     | 8    | load address      | The preferred load address for the image.                                                                                                                                                       |
| 48     | 8    | entry point       | The address of the first instruction to run relative to the load address.                                                                                                                       |
| 56     | 8    | destructor        | The address of the first instruction to run when unloading the module relative to the load address, if it is the same as the `entry point` there is no destructor.                              |
| 64     | 8    | flags             | The executable flags and characteristics, for specific flag values, see [[#Program Flags]].                                                                                                     |
| 72     | 8    | string table size | The number of the strings in the string table.                                                                                                                                                  |
| 80     | 8    | section count     | The amount of section entries, for more info see [[#Section Entry]].                                                                                                                            |
| 88     | 8    | symbol count      | The amount of symbol entries, for more info see [[#Symbol Entry]].                                                                                                                              |
| 96     | 8    | import count      | The amount of import entries, for more info see [[#Import Entry]].                                                                                                                              |
| 104    | 8    | relocation count  | The amount of relocation entries, for more info see [[#Relocation Entry]].                                                                                                                      |


### Program Flags
The Flags field contains flags that indicate attributes of the MELF module. The following flags are currently defined:

| Flag            | Value  | Description                                                                                                                |
| --------------- | ------ | -------------------------------------------------------------------------------------------------------------------------- |
| MSB             | 0x0001 | The binary data is in big endian.                                                                                          |
| FIXED_LOAD_ADDR | 0x0002 | Always load the binary at it's preferred load address, if the load address is not available the binary will not be loaded. |
| FORCE_INTEGRITY | 0x0004 | Integrity checks are enforced.                                                                                             |

## Section Entry

| Offset | Size | Field        | Description                                                                                                                                                                                 |
| ------ | ---- | ------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 0      | 8    | name         | A string index into the string table specifying the name of the section.                                                                                                                    |
| 8      | 8    | platform     | A string index into the string table specifying the platforms that this binary supports using the platform specifier format, for more about this format see [[#Platform Specifier Format]]. |
| 16     | 8    | flags        | The section flags and characteristics, For specific flag values, see [[#Section Flags]].                                                                                                    |
| 24     | 8    | load address | The address to load this section relative to the base address of the loaded image.                                                                                                          |
| 32     | 8    | offset       | The location of the data of this section starting from the start of the data segment in the binary, for more info see [[#Overview]].                                                        |
| 40     | 8    | real size    | The size of the section in the file.                                                                                                                                                        |
| 48     | 8    | load size    | The size of the data loaded in memory, if it is larger than the real size the rest will be zeroed, if it is smaller the section data will be cut.                                           |

### Section Flags
The Flags field contains flags that indicate attributes of the section. The following flags are currently defined:

| Flag    | Value  | Description                                         |
| ------- | ------ | --------------------------------------------------- |
| READ    | 0x0001 | Enable reading from the section.                    |
| WRITE   | 0x0002 | Enable writing from the section.                    |
| EXECUTE | 0x0004 | Enable executing from the section.                  |
| EXCLUDE | 0x0008 | Excludes the section from being loaded into memory. |

## Symbol Entry

| Offset | Size | Field   | Description                                                                                                                              |
| ------ | ---- | ------- | ---------------------------------------------------------------------------------------------------------------------------------------- |
| 0      | 8    | name    | A string index into the string table specifying the name of the symbol.                                                                  |
| 8      | 8    | section | An index into the section  table specifying the section that this symbol belong to, for more info about sections see [[#Section Entry]]. |
| 16     | 8    | offset  | The position of the start of the symbol data from the start of the section.                                                              |
| 24     | 8    | size    | The size of the data of the symbol in bytes.                                                                                             |
| 32     | 8    | flags   | The symbol flags and characteristics, For specific flag values, see [[#Symbol Flags]].                                                   |

### Symbol Flags
The Flags field contains flags that indicate attributes of the symbol. The following flags are currently defined:

| Flag    | Value  | Description                                                                                            |
| ------- | ------ | ------------------------------------------------------------------------------------------------------ |
| READ    | 0x0001 | Enable reading from the symbol.                                                                        |
| WRITE   | 0x0002 | Enable writing from the symbol.                                                                        |
| EXECUTE | 0x0004 | Enable executing from the symbol.                                                                      |
| EXPORT  | 0x0008 | Enable other melf modules to use this symbol.                                                          |
| IMPORT  | 0x0010 | The symbol is imported from another module using an import entry, for more info see [[#Import Entry]]. |
| WEAK    | 0x0020 | Marks the symbol as weak so it is overridable if there is another strong symbol with the same name.    |
## Import Entry

| Offset | Size | Field | Description                                                             |
| ------ | ---- | ----- | ----------------------------------------------------------------------- |
| 0      | 8    | path  | A string index into the string table specifying the path of the module. |

## Relocation Entry

| Offset | Size | Field   | Description                                                                                                                            |
| ------ | ---- | ------- | -------------------------------------------------------------------------------------------------------------------------------------- |
| 0      | 8    | section | An index into the section  table specifying which section is used for relocation, for more info about sections see [[#Section Entry]]. |
| 8      | 8    | symbol  | An index into the symbol table specifying which symbol is used for relocation, for more info about symbols see [[#Symbol Entry]].      |
| 16     | 8    | offset  | Offset in the section where the relocation needs to be applied.                                                                        |
| 24     | 2    | size    | The amount of bytes to change.                                                                                                         |
| 26     | 8    | flags   | The relocation entry flags and characteristics, For specific flag values, see [[#Relocation Flags]].                                   |
| 34     | 8    | addend  | Additional signed value to add to the symbol address.                                                                                  |
### Relocation Flags
The Flags field contains flags that indicate attributes of the relocation entry. The following flags are currently defined:

| Flag     | Value  | Description                                                                                                                                                                                 |
| -------- | ------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| ABSOLUTE | 0x0001 | The loader sets the address at the specified `offset` to the difference between the symbol's address and the address of the instruction plus the `addend`, , can't be used with `RELATIVE`. |
| RELATIVE | 0x0002 | The loader sets the address at the specified `offset` to the symbol's address plus the `addend`, can't be used with `ABSOLUTE`.                                                             |

# Trivia
- MELF is a typo of milf after I made the typo I thought about naming it MELF (My ELF) but when I was discussing relocation with chatgpt (3.5 to be exact) he hallucinated MELF (Minimal Executable and Linkable Format) so I went with that.