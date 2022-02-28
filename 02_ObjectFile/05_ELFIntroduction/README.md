# ELF入門

## ELFの型

|TypeName|*=32|*=64|Contents|
|:---:|:---|:---|:---|
|ELF*_Harf|uint16_t|uint16_t|unsigned 16bit|
|ELF*_Word|uint32_t|uint32_t|unsigned 32bit|
|ELF*_Sword|int32_t|int32_t|signed 32bit|
|ELF*_Xword|uint64_t|uint64_t|unsigned 64bit|
|ELF*_Sxword|int64_t|int64_t|signed 64bit|
|ELF*_Addr|uint32_t|uint64_t|Address|
|ELF*_Off|uint32_t|uint64_t|Offset|
|ELF*_Section|uint16_t|uint16_t|Section Index|
|ELF*_Versym|uint16_t|uint16_t|Version Symbol Information|

## ELF Header
ELFファイルのHeaderをチェックするにはreadelf -hを使用。
```console
$ readelf -h /bin/ls
ELF ヘッダ:
  マジック:   7f 45 4c 46 02 01 01 00 00 00 00 00 00 00 00 00 
  クラス:                            ELF64
  データ:                            2 の補数、リトルエンディアン
  バージョン:                        1 (current)
  OS/ABI:                            UNIX - System V
  ABI バージョン:                    0
  型:                                DYN (共有オブジェクトファイル)
  マシン:                            Advanced Micro Devices X86-64
  バージョン:                        0x1
  エントリポイントアドレス:               0x5850
  プログラムヘッダ始点:          64 (バイト)
  セクションヘッダ始点:          132000 (バイト)
  フラグ:                            0x0
  このヘッダのサイズ:                64 (バイト)
  プログラムヘッダサイズ:            56 (バイト)
  プログラムヘッダ数:                9
  セクションヘッダ:                  64 (バイト)
  セクションヘッダサイズ:            28
  セクションヘッダ文字列表索引:      27
```

情報の読み出し型については/usr/include/elf.hより取得可能。下記は64bitの場合でのELF Header。
EI_NIDENT = 16のため, e_identは16Byte使用.
```cpp
typedef struct
{
  unsigned char	e_ident[EI_NIDENT];	/* Magic number and other info */  => magic, class, data, version, OS/ABI, ABI Version
  Elf64_Half	e_type;			    /* Object file type */
  Elf64_Half	e_machine;		    /* Architecture */
  Elf64_Word	e_version;		    /* Object file version */
  Elf64_Addr	e_entry;		    /* Entry point virtual address */
  Elf64_Off	    e_phoff;		    /* Program header table file offset */
  Elf64_Off	    e_shoff;		    /* Section header table file offset */
  Elf64_Word	e_flags;		    /* Processor-specific flags */
  Elf64_Half	e_ehsize;		    /* ELF header size in bytes */
  Elf64_Half	e_phentsize;	    /* Program header table entry size */
  Elf64_Half	e_phnum;		    /* Program header table entry count */
  Elf64_Half	e_shentsize;	    /* Section header table entry size */
  Elf64_Half	e_shnum;		    /* Section header table entry count */
  Elf64_Half	e_shstrndx;		    /* Section header string table index */
} Elf64_Ehdr;
```

ELFファイルのheader => odでの16進数表示かつASCII文字出力で値を確認。  
1Byteずつ並んでいて, \177ELFの後, 次の2が「ELFCLASS64」, 次の1が「ELFDATA2LSB(Little Endian)」, 以降がELF Version, OS/ABI, ABI Versionと続く。
この一行が16Byteなので全てe_identを示す.
```console
$ od -An -tx1c /bin/ls | head -n2
  7f  45  4c  46  02  01  01  00  00  00  00  00  00  00  00  00
 177   E   L   F 002 001 001  \0  \0  \0  \0  \0  \0  \0  \0  \0
```

- 2Byteで表すe_typeは3で「ET_DYN(共有オブジェクトファイル)」になる。(自分の経験上, 現在のLinux実行ファイル系は大体DYNな認識)
- 2Byteで表すe_machineは62で「EM_X86_64(AMD x86_64)」
```console
$ od -An -tu2 /bin/ls | sed -n '2,2p'
     3    62     1     0 22608     0     0     0
```
```cpp
/* Legal values for e_type (object file type).  */
#define ET_NONE		0		/* No file type */
#define ET_REL		1		/* Relocatable file */
#define ET_EXEC		2		/* Executable file */
#define ET_DYN		3		/* Shared object file */
#define ET_CORE		4		/* Core file */

/* Legal values for e_machine (architecture).  */
#define EM_X86_64	62	/* AMD x86-64 architecture */
```

- 4Byteで表すe_versionは1で「EV_CURRENT」
```console
$ od -An -tu4 /bin/ls | sed -n '2,3p'
    4063235          1      22608          0
         64          0     132000          0
```
```cpp
/* Legal values for e_version (version).  */

#define EV_NONE		0		/* Invalid ELF version */
#define EV_CURRENT	1		/* Current version */
#define EV_NUM		2
```

- 8Byteで表すe_entryは0x5850アドレス
```console
$ od -An -tx8 /bin/ls | sed -n '2,2p'
 00000001003e0003 0000000000005850
```

- 8Byteで表すe_phoffは64Byte
- 8Byteで表すe_shoffは132000Byte
```console
$ od -An -tu8 /bin/ls | sed -n '3,3p'
                   64               132000
```

- 2Byteで表すe_flagsは0. Processor Specificなのだが, x86-64はelf.hからは読み取れなかった...
- 以降, 2Byte毎(Harfなのに1Byteではない?のがちょっとよくわからない...)に  
e_ehsize = 64, e_phentsize = 56, e_phnum = 9, e_shentsize = 64, e_shnum = 28, e_shstrndx = 27.
```console
$ od -An -tu2 /bin/ls | sed -n '4,4p'
     0     0    64    56     9    64    28    27
```

## Program Header

Program Headerをチェックするには readelf -lを使用。  
サイズはe_phentsize * e_phnum Byteある.
```console
$ readelf -l /bin/ls

Elf ファイルタイプは DYN (共有オブジェクトファイル) です
エントリポイント 0x5850
There are 9 program headers, starting at offset 64

プログラムヘッダ:
  タイプ        オフセット          仮想Addr           物理Addr
                 ファイルサイズ        メモリサイズ         フラグ 整列
  PHDR           0x0000000000000040 0x0000000000000040 0x0000000000000040
                 0x00000000000001f8 0x00000000000001f8  R E    0x8
  INTERP         0x0000000000000238 0x0000000000000238 0x0000000000000238
                 0x000000000000001c 0x000000000000001c  R      0x1
      [Requesting program interpreter: /lib64/ld-linux-x86-64.so.2]
  LOAD           0x0000000000000000 0x0000000000000000 0x0000000000000000
                 0x000000000001e6e8 0x000000000001e6e8  R E    0x200000
  LOAD           0x000000000001eff0 0x000000000021eff0 0x000000000021eff0
                 0x0000000000001278 0x0000000000002570  RW     0x200000
  DYNAMIC        0x000000000001fa38 0x000000000021fa38 0x000000000021fa38
                 0x0000000000000200 0x0000000000000200  RW     0x8
  NOTE           0x0000000000000254 0x0000000000000254 0x0000000000000254
                 0x0000000000000044 0x0000000000000044  R      0x4
  GNU_EH_FRAME   0x000000000001b1a0 0x000000000001b1a0 0x000000000001b1a0
                 0x0000000000000884 0x0000000000000884  R      0x4
  GNU_STACK      0x0000000000000000 0x0000000000000000 0x0000000000000000
                 0x0000000000000000 0x0000000000000000  RW     0x10
  GNU_RELRO      0x000000000001eff0 0x000000000021eff0 0x000000000021eff0
                 0x0000000000001010 0x0000000000001010  R      0x1

 セグメントマッピングへのセクション:
  セグメントセクション...
   00     
   01     .interp 
   02     .interp .note.ABI-tag .note.gnu.build-id .gnu.hash .dynsym .dynstr .gnu.version .gnu.version_r .rela.dyn .rela.plt .init .plt .plt.got .text .fini .rodata .eh_frame_hdr .eh_frame 
   03     .init_array .fini_array .data.rel.ro .dynamic .got .data .bss 
   04     .dynamic 
   05     .note.ABI-tag .note.gnu.build-id 
   06     .eh_frame_hdr 
   07     
   08     .init_array .fini_array .data.rel.ro .dynamic .got 
```
情報の読み出し型については/usr/include/elf.hより取得可能.下記は64bitの場合でのProgram Header.  
p_typeの値は別途定義があり, readelf上のタイプの箇所が対応。
```cpp
typedef struct
{
  Elf64_Word	p_type;			/* Segment type */
  Elf64_Word	p_flags;		/* Segment flags */
  Elf64_Off	p_offset;		/* Segment file offset */
  Elf64_Addr	p_vaddr;		/* Segment virtual address */
  Elf64_Addr	p_paddr;		/* Segment physical address */
  Elf64_Xword	p_filesz;		/* Segment size in file */
  Elf64_Xword	p_memsz;		/* Segment size in memory */
  Elf64_Xword	p_align;		/* Segment alignment */
} Elf64_Phdr;

/* Legal values for p_type (segment type).  */

#define	PT_NULL		0		/* Program header table entry unused */
#define PT_LOAD		1		/* Loadable program segment */
#define PT_DYNAMIC	2		/* Dynamic linking information */
#define PT_INTERP	3		/* Program interpreter */
#define PT_NOTE		4		/* Auxiliary information */
#define PT_SHLIB	5		/* Reserved */
#define PT_PHDR		6		/* Entry for header table itself */
#define PT_TLS		7		/* Thread-local storage segment */
#define	PT_NUM		8		/* Number of defined types */
#define PT_GNU_EH_FRAME	0x6474e550	/* GCC .eh_frame_hdr segment */
#define PT_GNU_STACK	0x6474e551	/* Indicates stack executability */
#define PT_GNU_RELRO	0x6474e552	/* Read-only after relocation */
```

## Section Header

Section Headerをチェックするには readelf -Sを使用。
サイズはe_shentsize * e_shnum Byteある.
```console
$ readelf -S /bin/ls
There are 28 section headers, starting at offset 0x203a0:

セクションヘッダ:
  [番] 名前              タイプ           アドレス          オフセット
       サイズ            EntSize          フラグ Link  情報  整列
  [ 0]                   NULL             0000000000000000  00000000
       0000000000000000  0000000000000000           0     0     0
  [ 1] .interp           PROGBITS         0000000000000238  00000238
       000000000000001c  0000000000000000   A       0     0     1
  [ 2] .note.ABI-tag     NOTE             0000000000000254  00000254
       0000000000000020  0000000000000000   A       0     0     4
  [ 3] .note.gnu.build-i NOTE             0000000000000274  00000274
       0000000000000024  0000000000000000   A       0     0     4
  [ 4] .gnu.hash         GNU_HASH         0000000000000298  00000298
       00000000000000ec  0000000000000000   A       5     0     8
  [ 5] .dynsym           DYNSYM           0000000000000388  00000388
       0000000000000df8  0000000000000018   A       6     1     8
  [ 6] .dynstr           STRTAB           0000000000001180  00001180
       0000000000000682  0000000000000000   A       0     0     1
  [ 7] .gnu.version      VERSYM           0000000000001802  00001802
       000000000000012a  0000000000000002   A       5     0     2
  [ 8] .gnu.version_r    VERNEED          0000000000001930  00001930
       0000000000000070  0000000000000000   A       6     1     8
  [ 9] .rela.dyn         RELA             00000000000019a0  000019a0
       0000000000001350  0000000000000018   A       5     0     8
  [10] .rela.plt         RELA             0000000000002cf0  00002cf0
       0000000000000a68  0000000000000018  AI       5    23     8
  [11] .init             PROGBITS         0000000000003758  00003758
       0000000000000017  0000000000000000  AX       0     0     4
  [12] .plt              PROGBITS         0000000000003770  00003770
       0000000000000700  0000000000000010  AX       0     0     16
  [13] .plt.got          PROGBITS         0000000000003e70  00003e70
       0000000000000018  0000000000000008  AX       0     0     8
  [14] .text             PROGBITS         0000000000003e90  00003e90
       00000000000124d9  0000000000000000  AX       0     0     16
  [15] .fini             PROGBITS         000000000001636c  0001636c
       0000000000000009  0000000000000000  AX       0     0     4
  [16] .rodata           PROGBITS         0000000000016380  00016380
       0000000000004e1d  0000000000000000   A       0     0     32
  [17] .eh_frame_hdr     PROGBITS         000000000001b1a0  0001b1a0
       0000000000000884  0000000000000000   A       0     0     4
  [18] .eh_frame         PROGBITS         000000000001ba28  0001ba28
       0000000000002cc0  0000000000000000   A       0     0     8
  [19] .init_array       INIT_ARRAY       000000000021eff0  0001eff0
       0000000000000008  0000000000000008  WA       0     0     8
  [20] .fini_array       FINI_ARRAY       000000000021eff8  0001eff8
       0000000000000008  0000000000000008  WA       0     0     8
  [21] .data.rel.ro      PROGBITS         000000000021f000  0001f000
       0000000000000a38  0000000000000000  WA       0     0     32
  [22] .dynamic          DYNAMIC          000000000021fa38  0001fa38
       0000000000000200  0000000000000010  WA       6     0     8
  [23] .got              PROGBITS         000000000021fc38  0001fc38
       00000000000003c8  0000000000000008  WA       0     0     8
  [24] .data             PROGBITS         0000000000220000  00020000
       0000000000000268  0000000000000000  WA       0     0     32
  [25] .bss              NOBITS           0000000000220280  00020268
       00000000000012e0  0000000000000000  WA       0     0     32
  [26] .gnu_debuglink    PROGBITS         0000000000000000  00020268
       0000000000000034  0000000000000000           0     0     4
  [27] .shstrtab         STRTAB           0000000000000000  0002029c
       0000000000000101  0000000000000000           0     0     1
Key to Flags:
  W (write), A (alloc), X (execute), M (merge), S (strings), I (info),
  L (link order), O (extra OS processing required), G (group), T (TLS),
  C (compressed), x (unknown), o (OS specific), E (exclude),
  l (large), p (processor specific)
```
情報の読み出し型については/usr/include/elf.hより取得可能.下記は64bitの場合でのSection Header.  
sh_typeの値は別途定義があり, readelf上のタイプの箇所が対応。
```cpp
typedef struct
{
  Elf64_Word	sh_name;		/* Section name (string tbl index) */
  Elf64_Word	sh_type;		/* Section type */
  Elf64_Xword	sh_flags;		/* Section flags */
  Elf64_Addr	sh_addr;		/* Section virtual addr at execution */
  Elf64_Off	    sh_offset;		/* Section file offset */
  Elf64_Xword	sh_size;		/* Section size in bytes */
  Elf64_Word	sh_link;		/* Link to another section */
  Elf64_Word	sh_info;		/* Additional section information */
  Elf64_Xword	sh_addralign;	/* Section alignment */
  Elf64_Xword	sh_entsize;		/* Entry size if section holds table */
} Elf64_Shdr;

/* Legal values for sh_type (section type).  */

#define SHT_NULL	        0		/* Section header table entry unused */
#define SHT_PROGBITS        1		/* Program data */
#define SHT_SYMTAB	        2		/* Symbol table */
#define SHT_STRTAB	        3		/* String table */
#define SHT_RELA	        4		/* Relocation entries with addends */
#define SHT_HASH	        5		/* Symbol hash table */
#define SHT_DYNAMIC	        6		/* Dynamic linking information */
#define SHT_NOTE	        7		/* Notes */
#define SHT_NOBITS	        8		/* Program space with no data (bss) */
#define SHT_REL		        9		/* Relocation entries, no addends */
#define SHT_SHLIB	        10		/* Reserved */
#define SHT_DYNSYM	        11		/* Dynamic linker symbol table */
#define SHT_INIT_ARRAY	    14		/* Array of constructors */
#define SHT_FINI_ARRAY	    15		/* Array of destructors */
#define SHT_PREINIT_ARRAY   16		/* Array of pre-constructors */
#define SHT_GROUP	        17		/* Section group */
#define SHT_SYMTAB_SHNDX    18		/* Extended section indeces */
#define	SHT_NUM		        19		/* Number of defined types.  */
#define SHT_GNU_ATTRIBUTES  0x6ffffff5	/* Object attributes.  */
#define SHT_GNU_HASH	    0x6ffffff6	/* GNU-style hash table.  */
#define SHT_GNU_LIBLIST	    0x6ffffff7	/* Prelink library list */
#define SHT_CHECKSUM	    0x6ffffff8	/* Checksum for DSO content.  */
#define SHT_GNU_verdef	    0x6ffffffd	/* Version definition section.  */
#define SHT_GNU_verneed	    0x6ffffffe	/* Version needs section.  */
#define SHT_GNU_versym	    0x6fffffff	/* Version symbol table.  */
```

## String Table

Section Header内の下記がStringTable
```console
  [番] 名前              タイプ           アドレス          オフセット
       サイズ            EntSize          フラグ Link  情報  整列
  [ 6] .dynstr           STRTAB           0000000000001180  00001180
       0000000000000682  0000000000000000   A       0     0     1
  [27] .shstrtab         STRTAB           0000000000000000  0002029c
       0000000000000101  0000000000000000           0     0     1
```

例えば.shstrtabを確認するなら, 0x2029c オフセット 0x101サイズのため, odで見るなら下記で見れる.
```console
$ od --skip-bytes 0x2029c --read-bytes 0x101 -tx1z /bin/ls
0401234 00 2e 73 68 73 74 72 74 61 62 00 2e 69 6e 74 65  >..shstrtab..inte<
0401254 72 70 00 2e 6e 6f 74 65 2e 41 42 49 2d 74 61 67  >rp..note.ABI-tag<
0401274 00 2e 6e 6f 74 65 2e 67 6e 75 2e 62 75 69 6c 64  >..note.gnu.build<
0401314 2d 69 64 00 2e 67 6e 75 2e 68 61 73 68 00 2e 64  >-id..gnu.hash..d<
0401334 79 6e 73 79 6d 00 2e 64 79 6e 73 74 72 00 2e 67  >ynsym..dynstr..g<
0401354 6e 75 2e 76 65 72 73 69 6f 6e 00 2e 67 6e 75 2e  >nu.version..gnu.<
0401374 76 65 72 73 69 6f 6e 5f 72 00 2e 72 65 6c 61 2e  >version_r..rela.<
0401414 64 79 6e 00 2e 72 65 6c 61 2e 70 6c 74 00 2e 69  >dyn..rela.plt..i<
0401434 6e 69 74 00 2e 70 6c 74 2e 67 6f 74 00 2e 74 65  >nit..plt.got..te<
0401454 78 74 00 2e 66 69 6e 69 00 2e 72 6f 64 61 74 61  >xt..fini..rodata<
0401474 00 2e 65 68 5f 66 72 61 6d 65 5f 68 64 72 00 2e  >..eh_frame_hdr..<
0401514 65 68 5f 66 72 61 6d 65 00 2e 69 6e 69 74 5f 61  >eh_frame..init_a<
0401534 72 72 61 79 00 2e 66 69 6e 69 5f 61 72 72 61 79  >rray..fini_array<
0401554 00 2e 64 61 74 61 2e 72 65 6c 2e 72 6f 00 2e 64  >..data.rel.ro..d<
0401574 79 6e 61 6d 69 63 00 2e 64 61 74 61 00 2e 62 73  >ynamic..data..bs<
0401614 73 00 2e 67 6e 75 5f 64 65 62 75 67 6c 69 6e 6b  >s..gnu_debuglink<
0401634 00                                               >.<
0401635
```

## Symbol Table

Symbolと値を対応させるテーブル. /bin/lsはstripされているので動的シンボルテーブルだけ存在している.
readelf -sで確認可能.
```console
$ readelf -s /bin/ls

Symbol table '.dynsym' contains 149 entries:
   番号:      値         サイズ タイプ  Bind   Vis      索引名
     0: 0000000000000000     0 NOTYPE  LOCAL  DEFAULT  UND 
     1: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND __ctype_toupper_loc@GLIBC_2.3 (2)
     2: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND __uflow@GLIBC_2.2.5 (3)
     3: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND getenv@GLIBC_2.2.5 (3)
     4: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND sigprocmask@GLIBC_2.2.5 (3)
     5: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND __snprintf_chk@GLIBC_2.3.4 (4)
     6: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND raise@GLIBC_2.2.5 (3)
     7: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND free@GLIBC_2.2.5 (3)
     8: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND abort@GLIBC_2.2.5 (3)
     9: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND __errno_location@GLIBC_2.2.5 (3)
    10: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND strncmp@GLIBC_2.2.5 (3)
    11: 0000000000000000     0 NOTYPE  WEAK   DEFAULT  UND _ITM_deregisterTMCloneTab
    12: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND localtime_r@GLIBC_2.2.5 (3)
    13: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND _exit@GLIBC_2.2.5 (3)
    14: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND strcpy@GLIBC_2.2.5 (3)
    15: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND __fpending@GLIBC_2.2.5 (3)
    16: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND isatty@GLIBC_2.2.5 (3)
    17: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND sigaction@GLIBC_2.2.5 (3)
    18: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND iswcntrl@GLIBC_2.2.5 (3)
    19: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND wcswidth@GLIBC_2.2.5 (3)
    20: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND localeconv@GLIBC_2.2.5 (3)
    21: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND mbstowcs@GLIBC_2.2.5 (3)
    22: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND readlink@GLIBC_2.2.5 (3)
    23: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND clock_gettime@GLIBC_2.17 (5)
    24: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND setenv@GLIBC_2.2.5 (3)
    25: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND textdomain@GLIBC_2.2.5 (3)
    26: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND fclose@GLIBC_2.2.5 (3)
    27: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND opendir@GLIBC_2.2.5 (3)
    28: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND getpwuid@GLIBC_2.2.5 (3)
    29: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND bindtextdomain@GLIBC_2.2.5 (3)
    30: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND dcgettext@GLIBC_2.2.5 (3)
    31: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND __ctype_get_mb_cur_max@GLIBC_2.2.5 (3)
    32: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND strlen@GLIBC_2.2.5 (3)
    33: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND __lxstat@GLIBC_2.2.5 (3)
    34: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND __stack_chk_fail@GLIBC_2.4 (6)
    35: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND getopt_long@GLIBC_2.2.5 (3)
    36: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND mbrtowc@GLIBC_2.2.5 (3)
    37: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND strchr@GLIBC_2.2.5 (3)
    38: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND getgrgid@GLIBC_2.2.5 (3)
    39: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND snprintf@GLIBC_2.2.5 (3)
    40: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND __overflow@GLIBC_2.2.5 (3)
    41: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND strrchr@GLIBC_2.2.5 (3)
    42: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND fgetfilecon
    43: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND gmtime_r@GLIBC_2.2.5 (3)
    44: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND lseek@GLIBC_2.2.5 (3)
    45: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND gettimeofday@GLIBC_2.2.5 (3)
    46: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND __assert_fail@GLIBC_2.2.5 (3)
    47: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND __strtoul_internal@GLIBC_2.2.5 (3)
    48: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND fnmatch@GLIBC_2.2.5 (3)
    49: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND memset@GLIBC_2.2.5 (3)
    50: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND fscanf@GLIBC_2.2.5 (3)
    51: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND ioctl@GLIBC_2.2.5 (3)
    52: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND getcwd@GLIBC_2.2.5 (3)
    53: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND close@GLIBC_2.2.5 (3)
    54: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND strspn@GLIBC_2.2.5 (3)
    55: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND closedir@GLIBC_2.2.5 (3)
    56: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND __libc_start_main@GLIBC_2.2.5 (3)
    57: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND memcmp@GLIBC_2.2.5 (3)
    58: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND _setjmp@GLIBC_2.2.5 (3)
    59: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND fputs_unlocked@GLIBC_2.2.5 (3)
    60: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND calloc@GLIBC_2.2.5 (3)
    61: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND lgetfilecon
    62: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND strcmp@GLIBC_2.2.5 (3)
    63: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND signal@GLIBC_2.2.5 (3)
    64: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND dirfd@GLIBC_2.2.5 (3)
    65: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND getpwnam@GLIBC_2.2.5 (3)
    66: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND __memcpy_chk@GLIBC_2.3.4 (4)
    67: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND sigemptyset@GLIBC_2.2.5 (3)
    68: 0000000000000000     0 NOTYPE  WEAK   DEFAULT  UND __gmon_start__
    69: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND memcpy@GLIBC_2.14 (7)
    70: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND getgrnam@GLIBC_2.2.5 (3)
    71: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND getfilecon
    72: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND tzset@GLIBC_2.2.5 (3)
    73: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND fileno@GLIBC_2.2.5 (3)
    74: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND tcgetpgrp@GLIBC_2.2.5 (3)
    75: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND __xstat@GLIBC_2.2.5 (3)
    76: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND readdir@GLIBC_2.2.5 (3)
    77: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND wcwidth@GLIBC_2.2.5 (3)
    78: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND malloc@GLIBC_2.2.5 (3)
    79: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND fflush@GLIBC_2.2.5 (3)
    80: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND nl_langinfo@GLIBC_2.2.5 (3)
    81: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND ungetc@GLIBC_2.2.5 (3)
    82: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND __fxstat@GLIBC_2.2.5 (3)
    83: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND strcoll@GLIBC_2.2.5 (3)
    84: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND mktime@GLIBC_2.2.5 (3)
    85: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND __freading@GLIBC_2.2.5 (3)
    86: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND fwrite_unlocked@GLIBC_2.2.5 (3)
    87: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND realloc@GLIBC_2.2.5 (3)
    88: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND stpncpy@GLIBC_2.2.5 (3)
    89: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND fdopen@GLIBC_2.2.5 (3)
    90: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND setlocale@GLIBC_2.2.5 (3)
    91: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND __printf_chk@GLIBC_2.3.4 (4)
    92: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND timegm@GLIBC_2.2.5 (3)
    93: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND strftime@GLIBC_2.2.5 (3)
    94: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND mempcpy@GLIBC_2.2.5 (3)
    95: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND memmove@GLIBC_2.2.5 (3)
    96: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND error@GLIBC_2.2.5 (3)
    97: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND open@GLIBC_2.2.5 (3)
    98: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND fseeko@GLIBC_2.2.5 (3)
    99: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND unsetenv@GLIBC_2.2.5 (3)
   100: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND strtoul@GLIBC_2.2.5 (3)
   101: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND __cxa_atexit@GLIBC_2.2.5 (3)
   102: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND wcstombs@GLIBC_2.2.5 (3)
   103: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND getxattr@GLIBC_2.3 (2)
   104: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND freecon
   105: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND gethostname@GLIBC_2.2.5 (3)
   106: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND sigismember@GLIBC_2.2.5 (3)
   107: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND exit@GLIBC_2.2.5 (3)
   108: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND fwrite@GLIBC_2.2.5 (3)
   109: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND __fprintf_chk@GLIBC_2.3.4 (4)
   110: 0000000000000000     0 NOTYPE  WEAK   DEFAULT  UND _ITM_registerTMCloneTable
   111: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND fflush_unlocked@GLIBC_2.2.5 (3)
   112: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND mbsinit@GLIBC_2.2.5 (3)
   113: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND iswprint@GLIBC_2.2.5 (3)
   114: 0000000000000000     0 FUNC    WEAK   DEFAULT  UND __cxa_finalize@GLIBC_2.2.5 (3)
   115: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND sigaddset@GLIBC_2.2.5 (3)
   116: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND __ctype_tolower_loc@GLIBC_2.3 (2)
   117: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND __ctype_b_loc@GLIBC_2.3 (2)
   118: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND __sprintf_chk@GLIBC_2.3.4 (4)
   119: 0000000000220280     8 OBJECT  GLOBAL DEFAULT   25 __progname@GLIBC_2.2.5 (3)
   120: 00000000002201e0     4 OBJECT  GLOBAL DEFAULT   24 ls_mode
   121: 000000000001636c     0 FUNC    GLOBAL DEFAULT   15 _fini
   122: 0000000000220290     4 OBJECT  GLOBAL DEFAULT   25 optind@GLIBC_2.2.5 (3)
   123: 0000000000003758     0 FUNC    GLOBAL DEFAULT   11 _init
   124: 00000000002202a8     8 OBJECT  WEAK   DEFAULT   25 program_invocation_name@GLIBC_2.2.5 (3)
   125: 0000000000220268     0 NOTYPE  GLOBAL DEFAULT   25 __bss_start
   126: 000000000001aec0    47 OBJECT  GLOBAL DEFAULT   16 version_etc_copyright
   127: 00000000002201e8     8 OBJECT  GLOBAL DEFAULT   24 Version
   128: 0000000000016380     4 OBJECT  GLOBAL DEFAULT   16 _IO_stdin_used
   129: 000000000021f9e0    88 OBJECT  GLOBAL DEFAULT   21 quoting_style_args
   130: 0000000000221560     0 NOTYPE  GLOBAL DEFAULT   25 _end
   131: 00000000002201f8     4 OBJECT  GLOBAL DEFAULT   24 exit_failure
   132: 00000000002202a8     8 OBJECT  GLOBAL DEFAULT   25 __progname_full@GLIBC_2.2.5 (3)
   133: 0000000000220200    56 OBJECT  GLOBAL DEFAULT   24 quote_quoting_options
   134: 00000000002201f0     8 OBJECT  GLOBAL DEFAULT   24 argmatch_die
   135: 0000000000015690    42 FUNC    GLOBAL DEFAULT   14 _obstack_memory_used
   136: 0000000000220260     8 OBJECT  GLOBAL DEFAULT   24 obstack_alloc_failed_hand
   137: 00000000000154b0    17 FUNC    GLOBAL DEFAULT   14 _obstack_begin
   138: 0000000000220268     0 NOTYPE  GLOBAL DEFAULT   24 _edata
   139: 00000000002202c0     8 OBJECT  GLOBAL DEFAULT   25 stderr@GLIBC_2.2.5 (3)
   140: 0000000000015620   106 FUNC    GLOBAL DEFAULT   14 _obstack_free
   141: 0000000000220280     8 OBJECT  WEAK   DEFAULT   25 program_invocation_short_@GLIBC_2.2.5 (3)
   142: 00000000000155e0    56 FUNC    GLOBAL DEFAULT   14 _obstack_allocated_p
   143: 00000000002202a0     8 OBJECT  GLOBAL DEFAULT   25 optarg@GLIBC_2.2.5 (3)
   144: 00000000000154d0    21 FUNC    GLOBAL DEFAULT   14 _obstack_begin_1
   145: 000000000001ab60    40 OBJECT  GLOBAL DEFAULT   16 quoting_style_vals
   146: 00000000000154f0   235 FUNC    GLOBAL DEFAULT   14 _obstack_newchunk
   147: 0000000000220288     8 OBJECT  GLOBAL DEFAULT   25 stdout@GLIBC_2.2.5 (3)
   148: 0000000000221400     8 OBJECT  GLOBAL DEFAULT   25 program_name
```

これら動的シンボルテーブルはELFヘッダから読み取ることができ, .dynsymセクションから読み取れる.
```console
  [番] 名前              タイプ           アドレス          オフセット
       サイズ            EntSize          フラグ Link  情報  整列
  [ 5] .dynsym           DYNSYM           0000000000000388  00000388
       0000000000000df8  0000000000000018   A       6     1     8
```
```console
$ od --skip-bytes 0x388 --read-bytes 0xdf8 -tx1z /bin/ls
0001610 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  >................<
0001630 00 00 00 00 00 00 00 00 43 04 00 00 12 00 00 00  >........C.......<
0001650 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  >................<
0001670 bb 02 00 00 12 00 00 00 00 00 00 00 00 00 00 00  >................<
0001710 00 00 00 00 00 00 00 00 e5 02 00 00 12 00 00 00  >................<
0001730 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  >................<
0001750 3c 01 00 00 12 00 00 00 00 00 00 00 00 00 00 00  ><...............<
0001770 00 00 00 00 00 00 00 00 4b 03 00 00 12 00 00 00  >........K.......<
...(略)
```

Symbol Tableの構造. st_infoの下位4bitがSTT, 上位4bitがSTB側の意味を示す.  
st_shndxは「Special section indices」のコメントにある情報がある.
```cpp
typedef struct
{
  Elf64_Word	st_name;		/* Symbol name (string tbl index) */
  unsigned char	st_info;		/* Symbol type and binding */
  unsigned char st_other;		/* Symbol visibility */
  Elf64_Section	st_shndx;		/* Section index */
  Elf64_Addr	st_value;		/* Symbol value */
  Elf64_Xword	st_size;		/* Symbol size */
} Elf64_Sym;

/* Legal values for ST_TYPE subfield of st_info (symbol type).  */

#define STT_NOTYPE	0		/* Symbol type is unspecified */
#define STT_OBJECT	1		/* Symbol is a data object */
#define STT_FUNC	2		/* Symbol is a code object */
#define STT_SECTION	3		/* Symbol associated with a section */
#define STT_FILE	4		/* Symbol's name is file name */
#define STT_COMMON	5		/* Symbol is a common data object */
#define STT_TLS		6		/* Symbol is thread-local data object*/
#define	STT_NUM		7		/* Number of defined types.  */

/* Legal values for ST_BIND subfield of st_info (symbol binding).  */

#define STB_LOCAL	0		/* Local symbol */
#define STB_GLOBAL	1		/* Global symbol */
#define STB_WEAK	2		/* Weak symbol */
#define	STB_NUM		3		/* Number of defined types.  */

/* Special section indices.  */

#define SHN_UNDEF	0		/* Undefined section */
#define SHN_LORESERVE	0xff00		/* Start of reserved indices */
#define SHN_LOPROC	0xff00		/* Start of processor-specific */
#define SHN_BEFORE	0xff00		/* Order section before all others
					   (Solaris).  */
#define SHN_AFTER	0xff01		/* Order section after all others
					   (Solaris).  */
#define SHN_HIPROC	0xff1f		/* End of processor-specific */
#define SHN_LOOS	0xff20		/* Start of OS-specific */
#define SHN_HIOS	0xff3f		/* End of OS-specific */
#define SHN_ABS		0xfff1		/* Associated symbol is absolute */
#define SHN_COMMON	0xfff2		/* Associated symbol is common */
#define SHN_XINDEX	0xffff		/* Index is in extra table.  */
#define SHN_HIRESERVE	0xffff		/* End of reserved indices */
```
例えば, 下記はst_nameが0x013c(little endianのため).
```console
0001750 3c 01 00 00 12 00 00 00 00 00 00 00 00 00 00 00  ><...............<
```

.dynstrセクションのstring tableが0x1180オフセットにあり, 
```console
  [番] 名前              タイプ           アドレス          オフセット
       サイズ            EntSize          フラグ Link  情報  整列
  [ 6] .dynstr           STRTAB           0000000000001180  00001180
       0000000000000682  0000000000000000   A       0     0     1
```

0x013cを足した0x12bcを見ると, sigprocmaskが見つかる.
```console
$ od --skip-bytes 0x12bc --read-bytes 16 -tx1z /bin/ls
0011274 73 69 67 70 72 6f 63 6d 61 73 6b 00 5f 5f 73 74  >sigprocmask.__st<
0011314
```

readelf上では下記に対応するもの.
```console
     4: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND sigprocmask@GLIBC_2.2.5 (3)
```

## Relocation情報

SHT_RELA or SHT_RELタイプセクションは再配置情報を持つ.  
r_offsetはリロケーションを行うべき場所を示すオフセット.  
Relaの場合にはリロケーションする場合に常に加算する値r_addendを持つ.
```cpp
typedef struct
{
  Elf64_Addr	r_offset;		/* Address */
  Elf64_Xword	r_info;			/* Relocation type and symbol index */
  Elf64_Sxword	r_addend;		/* Addend */
} Elf64_Rela;

typedef struct
{
  Elf64_Addr	r_offset;		/* Address */
  Elf64_Xword	r_info;			/* Relocation type and symbol index */
} Elf64_Rel;
```
