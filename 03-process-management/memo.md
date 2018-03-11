# 第 3 章：プロセス管理

## fork()

1. 子プロセスのメモリ領域を確保し、親プロセスのメモリをコピー
2. fork() は親プロセスには子プロセスの `pid`を、子プロセスには `0` を返す

## execve()

1. 実行ファイルからメモリマップに必要な情報を読み出す
2. 現在のプロセスのメモリを新しいプロセスのデータで上書き
    - 実行ファイルの情報に基いてメモリ上にコード、データを展開
3. 新しいプロセスの最初の命令（エントリポイント）から実行

- `Entry point address` が `/bin/sleep` のエントリポイント

```
$ readelf -h /bin/sleep
ELF Header:
  Magic:   7f 45 4c 46 02 01 01 00 00 00 00 00 00 00 00 00
  Class:                             ELF64
  Data:                              2's complement, little endian
  Version:                           1 (current)
  OS/ABI:                            UNIX - System V
  ABI Version:                       0
  Type:                              EXEC (Executable file)
  Machine:                           Advanced Micro Devices X86-64
  Version:                           0x1
  Entry point address:               0x401760
  Start of program headers:          64 (bytes into file)
  Start of section headers:          29552 (bytes into file)
  Flags:                             0x0
  Size of this header:               64 (bytes)
  Size of program headers:           56 (bytes)
  Number of program headers:         9
  Size of section headers:           64 (bytes)
  Number of section headers:         29
  Section header string table index: 28
```

- `.text` がコード領域、`.data` がデータ領域
- `Address` がメモリマップ開始アドレス
- `Offset` がファイル内オフセット
- `Size` がサイズ

```
$ readelf -S /bin/sleep
There are 29 section headers, starting at offset 0x7370:

Section Headers:
  [Nr] Name              Type             Address           Offset
       Size              EntSize          Flags  Link  Info  Align
  [ 0]                   NULL             0000000000000000  00000000
       0000000000000000  0000000000000000           0     0     0
  [ 1] .interp           PROGBITS         0000000000400238  00000238
       000000000000001c  0000000000000000   A       0     0     1
  [ 2] .note.ABI-tag     NOTE             0000000000400254  00000254
       0000000000000020  0000000000000000   A       0     0     4
  [ 3] .note.gnu.build-i NOTE             0000000000400274  00000274
       0000000000000024  0000000000000000   A       0     0     4
  [ 4] .gnu.hash         GNU_HASH         0000000000400298  00000298
       0000000000000044  0000000000000000   A       5     0     8
  [ 5] .dynsym           DYNSYM           00000000004002e0  000002e0
       00000000000005b8  0000000000000018   A       6     1     8
  [ 6] .dynstr           STRTAB           0000000000400898  00000898
       0000000000000282  0000000000000000   A       0     0     1
  [ 7] .gnu.version      VERSYM           0000000000400b1a  00000b1a
       000000000000007a  0000000000000002   A       5     0     2
  [ 8] .gnu.version_r    VERNEED          0000000000400b98  00000b98
       0000000000000060  0000000000000000   A       6     1     8
  [ 9] .rela.dyn         RELA             0000000000400bf8  00000bf8
       00000000000000a8  0000000000000018   A       5     0     8
  [10] .rela.plt         RELA             0000000000400ca0  00000ca0
       00000000000004c8  0000000000000018  AI       5    24     8
  [11] .init             PROGBITS         0000000000401168  00001168
       000000000000001a  0000000000000000  AX       0     0     4
  [12] .plt              PROGBITS         0000000000401190  00001190
       0000000000000340  0000000000000010  AX       0     0     16
  [13] .plt.got          PROGBITS         00000000004014d0  000014d0
       0000000000000008  0000000000000000  AX       0     0     8
  [14] .text             PROGBITS         00000000004014e0  000014e0
       0000000000003319  0000000000000000  AX       0     0     16
  [15] .fini             PROGBITS         00000000004047fc  000047fc
       0000000000000009  0000000000000000  AX       0     0     4
  [16] .rodata           PROGBITS         0000000000404820  00004820
       0000000000000e3a  0000000000000000   A       0     0     32
  [17] .eh_frame_hdr     PROGBITS         000000000040565c  0000565c
       000000000000025c  0000000000000000   A       0     0     4
  [18] .eh_frame         PROGBITS         00000000004058b8  000058b8
       0000000000000c7c  0000000000000000   A       0     0     8
  [19] .init_array       INIT_ARRAY       0000000000606e10  00006e10
       0000000000000008  0000000000000000  WA       0     0     8
  [20] .fini_array       FINI_ARRAY       0000000000606e18  00006e18
       0000000000000008  0000000000000000  WA       0     0     8
  [21] .jcr              PROGBITS         0000000000606e20  00006e20
       0000000000000008  0000000000000000  WA       0     0     8
  [22] .dynamic          DYNAMIC          0000000000606e28  00006e28
       00000000000001d0  0000000000000010  WA       6     0     8
  [23] .got              PROGBITS         0000000000606ff8  00006ff8
       0000000000000008  0000000000000008  WA       0     0     8
  [24] .got.plt          PROGBITS         0000000000607000  00007000
       00000000000001b0  0000000000000008  WA       0     0     8
  [25] .data             PROGBITS         00000000006071c0  000071c0
       0000000000000074  0000000000000000  WA       0     0     32
  [26] .bss              NOBITS           0000000000607240  00007234
       00000000000001c0  0000000000000000  WA       0     0     32
  [27] .gnu_debuglink    PROGBITS         0000000000000000  00007234
       0000000000000034  0000000000000000           0     0     1
  [28] .shstrtab         STRTAB           0000000000000000  00007268
       0000000000000102  0000000000000000           0     0     1
Key to Flags:
  W (write), A (alloc), X (execute), M (merge), S (strings), l (large)
  I (info), L (link order), G (group), T (TLS), E (exclude), x (unknown)
  O (extra OS processing required) o (OS specific), p (processor specific)
```

### fork & exec

- 親プロセスから子プロセスを作り、子プロセスを書き換えて実行する
- `bash` が `/bin/echo` を呼ぶときなど

## exit()

- プロセスに割り当てていたメモリを回収
- 標準 C ライブラリの exit()
- main() 関数から復帰した時も同様
