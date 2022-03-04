# ldd

共有ライブラリを利用する実行ファイルは実行時に必要な別の共有ライブラリが何かという情報を持つ.  
ELFの「動的セクション」にNEEDEDで記録されていて, objdumpやreadelfで確認可能.  
```console
$ objdump -p /bin/ls
/bin/ls:     ファイル形式 elf64-x86-64
プログラムヘッダ:
    (略)

動的セクション:
  NEEDED               libselinux.so.1
  NEEDED               libc.so.6
  INIT                 0x0000000000003758
　(略)

バージョン参照:
  (略)
```
```console
$ readelf -d /bin/ls
Dynamic section at offset 0x1fa38 contains 28 entries:
  タグ        タイプ                       名前/値
 0x0000000000000001 (NEEDED)             共有ライブラリ: [libselinux.so.1]
 0x0000000000000001 (NEEDED)             共有ライブラリ: [libc.so.6]
 0x000000000000000c (INIT)               0x3758
 (略)
```
この/bin/lsはNEEDEDで指定しているライブラリだけでなく, これらのライブラリが必要としている別のライブラリも必要.  
ライブラリは/usr/lib, /lib, LD_LIBRARY_PATH, /etc/ld.so.cacheの情報からSONAMEに対応するファイルを探索する.  

objdumpやreadelfはこの依存ライブラリを全て辿るのは出来ないので, 探るにはlddコマンドを使う.  
(lddはshell scriptであり, 実行ファイルにはLD_TRACE_LOADED_OBJECTS=1の状態でプログラム実行,  
 共有ライブラリの場合はLD_TRACE_LOADED_OBJECTS=1の状態でldを実行する.)  
```
$ ldd /bin/ls
        linux-vdso.so.1 (0x00007ffe7e1b6000)
        libselinux.so.1 => /lib/x86_64-linux-gnu/libselinux.so.1 (0x00007f8f3dd44000)
        libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6 (0x00007f8f3d953000)
        libpcre.so.3 => /lib/x86_64-linux-gnu/libpcre.so.3 (0x00007f8f3d6e1000)
        libdl.so.2 => /lib/x86_64-linux-gnu/libdl.so.2 (0x00007f8f3d4dd000)
        /lib64/ld-linux-x86-64.so.2 (0x00007f8f3e18e000)
        libpthread.so.0 => /lib/x86_64-linux-gnu/libpthread.so.0 (0x00007f8f3d2be000)
```
